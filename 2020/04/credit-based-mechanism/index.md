# 区块链实验7-信誉机制设计


J. Huang, L. Kong, G. Chen, M.-Y. Wu, X. Liu, and P. Zeng, “Towards Secure Industrial IoT: Blockchain System With Credit-Based Consensus Mechanism,” *IEEE Trans. Ind. Inf.*, vol. 15, no. 6, pp. 3680–3689, Jun. 2019, doi: [10.1109/TII.2019.2903342](https://doi.org/10.1109/TII.2019.2903342).

整体思路来自于上述论文的 Section III，B. Credit-Based PoW Mechanism，所作的调整会在本文详细解释。

## 1. 信誉机制

作者设计了一个基于信誉的 PoW 共识机制来取得效率与安全的平衡。首先为节点 $i$ 设置一个信誉值属性 $Cr_i$，该值会随着节点的行为实时的变化。正常的行为，如遵守系统规则发送交易，会随着时间的推移使信誉值逐步增加，与之相反，节点产生异常行为会导致信誉值下降。PoW机制的难度根据每个节点的信誉值自调整，信誉值越低，运行 PoW 算法花费的时间越长。因此，诚实的节点消耗的资源更少，恶意节点攻击所需的花费更多。

这里首先声明两种系统中可能存在的恶意行为

1. Lazy Tips：懒惰的节点指那些总是验证固定的以前的交易，而不去验证最新的交易的节点。例如，恶意实体可以通过发出许多验证固定交易对的交易来人为地扩大[提示]^(tips)的数量。这会使其它节点有更高的概率选中这些提示，而丢弃属于诚实节点的提示
2. Double-spending：通过在前一次花费被验证之前提交多个交易，恶意节点希望将一枚代币花费两次或多次，这就是双花问题。尽管这样的行为会被共识机制检测到并撤销，但它降低了系统效率，因为其它相关的交易也会被撤销重新执行。

### 1.1 信誉值定义

因此，根据节点 $i$ 的行为，我们将其信誉值 $Cr_i$ 划分为两部分，公式如下
$$
Cr_i = \lambda_1 Cr_i^P + \lambda_2 Cr_i^N
$$
其中 $Cr_i^P$ 代表正面影响部分，$Cr_i^P$ 代表负面影响部分。$\lambda_1$ 和 $\lambda_2$ 分别代表各部分的权重系数，调节这两个值就可以调整两部分所占权重，比如，如果我们想要严格的惩罚策略，应该令 $\lambda_2$ 更大一点。

$Cr_i^P$ 与节点 $i$ 单位时间内正常的交易数量成正相关，即通过节点活跃程度定义，表示如下
$$
Cr_i^P = \frac{\sum_{k=1}^{n_i} \omega_k} {\Delta T}
$$
其中 $n_i$ 代表节点 $i$ 在最近的单位时间内有效交易的数量，$\Delta T$ 代表单位时间，$\omega_k$ 代表第 $k$ 个交易的权重，交易的权重指的是该交易被验证的次数。也就是说，如果节点 $i$ 在一段时间内保持活跃，$Cr_i^P$ 将根据活跃程度不断调整，保证活跃节点可以使用更少的算力更快地发布交易。如果节点 $i$ 在一段时间内没有发布交易，就认为它是不活跃的，甚至是不可信节点，所以系统不会为它降低 PoW 的难度，即 $Cr_i^P = 0$。

$Cr_i^N$ 与节点 $i$ 的恶意行为数量成负相关，可以表示为
$$
Cr_i^N = -\sum_{k=1}^{m_i} \alpha(\beta) · \frac{\Delta T}{t-t_k}
$$
其中 $m_i$ 表示节点 $i$ 的恶意行为总数，$t$ 表示当前时间，$t_k$ 表示节点 $i$ 造成的第 $k$ 个恶意行为的时间点，$\alpha(\beta)$ 表示恶意行为 $\beta$ 的惩罚系数，该系数定义如下，其中 $\alpha_l$ 和 $\alpha_d$ 可以根据对恶意行为敏感度的要求进行调整。
$$
\alpha(\beta) = \begin{cases} \alpha_l&\text{if β is lazy tips behavior;}  \\\ \alpha_d & \text{if β is double-spending behavior} \end{cases}
$$
从$Cr_i^N$ 的公式中我们可以发现，随着时间的推移，恶意行为对节点的影响在逐渐减小，但不同于 $Cr_i^P$，它无法减小到0，也就是完全消除。当一个恶意行为发生的时候，$Cr_i^N$ 的绝对值会很大，由于 PoW 难度巨大，攻击将无法持续，通过这种方式我们可以及时阻止恶意行为。

该机制正常运行的需求是我们可以获取每个节点相关的所有交易，这样就可以计算出交易权重 $\omega$ 和 恶意行为记录 $\alpha(\beta)$，从而可以独立地计算出 $Cr_i^P$ 和 $Cr_i^N$，最终得到信誉值。作者在论文中将信誉值与 PoW 难度关联，具体来说，这两种成反比，定义公式为 $Cr_i = \delta \frac 1{D_i}$，其中 $D_i$ 为节点 $i$ 的 PoW 难度，$\delta$ 为比例系数。这样，信誉值高的难度低，信誉值低的难度高，难度的调整通过控制前缀0的最小长度完成，整个系统得以实现。

### 1.2 参数设置

具体的实验中以上公式中的相关参数如何设置，作者给出了一些描述。交易权重 $\omega$ 可以直接计算，两个权重系数设置为 $\lambda_1 = 1,\lambda_2 = 0.5$，因为 $Cr_i^N$ 的值可能相对比较大，如果想要更严厉的惩罚措施，$\lambda_2$ 可以设置的更大。考虑到 IIoT 系统的请求频率，单位时间设置为 $\Delta T = 30s$，一个不是太长的间隔。对于 lazy tips，设置 $\alpha(\beta) = 0.5$，对于 double-spending，设置 $\alpha(\beta) = 1$，因为双花对系统造成的损害更严重。

 ## 2. 相关调整

我们在方案中使用该值的目的是对恶意行为做出惩罚，惩罚的方式是阻塞一段固定的时间，在这段时间内设备的所有访问请求都被拒绝。因为没有合适的方式对遵守规则的设备进行奖励，即时我们可以通过被通过的访问控制请求的数量定义正面影响 $Cr_i^P$，最后也无法产生任何作用。

### 2.1 惩罚函数

因此我们只取信誉值的负面影响部分 $Cr_i^N$，并将其定义为惩罚函数。惩罚函数的值应与恶意行为的总数相关，其随着时间的推移影响逐渐减小，但不会变为0。对于惩罚函数的更新，应该每次产生恶意行为都进行一次，而非间隔固定的时间计算一次，最终惩罚函数重新定义为
$$
Cr_i^N = \sum_{k=1}^{m_i} \alpha(\beta) · \frac1{t-t_k}
$$
其中 $m_i$ 表示设备 $i$ 的恶意行为总数，$t$ 表示当前时间，$t_k$ 表示设备 $i$ 造成的第 $k$ 个恶意行为的时间点，$\alpha(\beta)$ 表示恶意行为 $\beta$ 的惩罚系数，该系数定义如下，其中 $\alpha_1$ 和 $\alpha_2$ 可以根据对恶意行为敏感度的要求进行调整。
$$
\alpha(\beta) = \begin{cases} \alpha_1&\text{如果 β 代表短时间发起大量请求 ;}  \\\ \alpha_2 & \text{如果 β 代表企图修改无权限的属性或策略 } \end{cases}
$$
该值应与阻塞时间 T 成正比例或指数函数关系，值越大，阻塞时间越长，意味着设备在更长的一段时间内所有访问请求都不会被通过。设备应拥有一个惩罚终止时间属性，每次发起访问控制都会被 Object 判定，如果当前时间小于终止时间，则直接拒绝请求，如果大于终止时间，则进行其它判定。

### 2.2 信誉函数

如果我们能找到一种合适的方式对遵守规则的节点进行奖励，可以定义信誉值函数
$$
Cr_i = \lambda_1 Cr_i^P - \lambda_2 Cr_i^N
$$
其中 $Cr_i^N$ 是我们上面定义的惩罚函数，代表负面影响部分，$\lambda_1$ 和 $\lambda_2$ 分别代表各部分的权重系数，调节这两个值就可以调整两部分所占权重。$Cr_i^P$ 代表正面影响部分，应与合法的，也就是被通过的访问控制请求数量有关，定义如下，其中 $N$ 为有效请求数量
$$
Cr_i^P = \delta_1 N
$$
另外，可以根据访问请求的不同情况设置不同的权重

1. 如果根据访问操作的不同（读取、执行）设置权重，函数调整如下，其中 $\omega_k$ 代表第 $k$ 中操作的权重，$n_k$ 代表第 $k$ 种操作的数量
   $$
   Cr_i^P = \delta_1 \sum_{k=1}^2 \omega_k n_k
   $$

2. 如果根据访问对象的不同设置权重，函数调整如下，$\omega$ 和 $n$ 是不同对象的权重和收到的请求数量
   $$
   Cr_i^P = \delta_1 \sum_{k=1}^n \omega_k n_k
   $$

最后，函数的值不应当与设备的活跃程度有关，意为，通过的访问请求数量多的设备不一定信誉值高，因为有的设备可能需要发起更多的请求，有的设备不需要。因此我们将信誉值调整为增量形式
$$
NewCr_i = OldCr_i + \lambda_1 Cr_i^P - \lambda_2 Cr_i^N
$$
这样，正面和负面影响都应只计算上一次信誉值更新之后的行为，同时还应考虑：如果设备一直遵守规则，信誉值会保持不断增长，最终可能导致超限。

之所以不采取原论文那种单位时间内请求数量的方式计算，是考虑到设备可能在某段时间一次请求都不发起，这种情况应该不对设备的信誉值产生影响。