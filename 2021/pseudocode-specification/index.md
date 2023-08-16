# 科研基础3-伪代码规范


论文的编辑要插入两段伪代码，这里总结一下伪代码书写用到的 LaTeX 包和书写规范。

<!--more-->

## 1. 伪代码规范

伪代码是一种接近自然语言的算法描述形式，其目的是在不涉及具体实现（各种编程语言）的情况下将算法的流程和含义清楚的表达出来，因此它没有一个统一的规范，有的仅仅是在长期的实践过程中形成的一些约定俗成的表达样式。下图是一个简单的例子[^1]，但已经包含了大多数主要元素

[^1]:P. Wang, Y. Yue, W. Sun and J. Liu, "An Attribute-Based Distributed Access Control for Blockchain-enabled IoT," *2019 International Conference on Wireless and Mobile Computing, Networking and Communications (WiMob)*, Barcelona, Spain, 2019, pp. 1-6, doi: 10.1109/WiMOB.2019.8923232.

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20210118_示例伪代码.png" alt="示例伪代码" style="zoom:80%;" />

首先需要一个**标题**来描述整个算法，一般还会有一个与之一起的编号。在上图中，算法的标题为「Acesscontrol algorithm flow」，编号为「Algorithm 1」。标题与编号一般位于算法顶部，但也有人放在底部，编号多按全文的算法总数进行索引，但也可以按章节分别进行索引。

其次，在正式的算法流之前需要声明**输入和输出**。通常使用关键字 Input 和 Output 来声明，但也有人使用关键字 Data 和 Result。

顺序、选择与循环结构是算法的主体。通常，不同的程序块使用缩进来保持结构清晰，但也有不少人使用垂直连接线进行进一步划分，如上图。另外，和编程语言类似，伪代码中的选择和循环使用 if-then-else、while-do 和 for-do 等关键字和结构来描述[^2]，举例如下，其中，每个程序块结束的 end 关键字有人选择写，也有人选择不写。

[^2]:[algorithm2e package document](http://mirrors.ctan.org/macros/latex/contrib/algorithm2e/doc/algorithm2e.pdf)

![算法流程结构](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20210118_算法流程结构.png)

赋值操作一般使用左箭头「<—」表示。A[i] 用来表示数组 A 的第 i 个元素，A[1…j] 则表示下标从 1 到 j 的子数组；函数调用使用函数名+传入参数的形式；返回值使用 return 关键字，这些都和常规编程语言相同。需要注意的是，未声明而使用的变量都可以视为算法内的局部变量，如果是全局变量则需要进行解释，可以在上下文中，也可以使用注释；注释的形式也和传统语言相同，使用 // 或 /*……\*/。

算法整体通常使用三线框包围，但也有少部分人使用一个完整的框。

伪代码的语句一般不需要在末尾使用分号，但行首通常会添加行号。

## 2. LaTeX包

latex 书写伪代码主要有三种排版格式：algorithm+algorithmic、algorithm+algorithmicx 以及 algorithm2e[^3]。我们使用 [algorithm2e](https://www.ctan.org/pkg/algorithm2e)，它提供了垂直连接线，可以去掉 end 关键字，而且写起来更像编程语言，用着非常舒服。

[^3]:Joe_WQ，latex 中的 algorithm 环境，简书，https://www.jianshu.com/p/fe9ba3c2424d

引入 algorithm2e 包使用如下语句

```
\usepackage[options]{algorithm2e}
```

几个重要的 options 如下

1. ruled：让标题显示在上面，默认会显示到最下面；
2. vlined：默认启用垂直连接线；
3. linesnumbered：让算法显示行号，不包括 input 和 output 部分；
4. noend：程序块结束不打印 end。

常用命令如下[^4]

[^4]:熏风初入弦，用 LaTeX 优雅地书写伪代码—Algorithm2e 简明指南，知乎，https://zhuanlan.zhihu.com/p/166418214

| 命令                           | 含义                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| \caption{}                     | 插入标题                                                     |
| \KwIn{输入信息}                | 效果为：“In：输入信息”                                       |
| \KwOut{输出信息}               | 效果为：“Out：输出信息”                                      |
| \For{条件}{循环语句}           | for 条件 do<br />    循环语句<br />end                       |
| \If{条件}{肯定语句}            | if 条件 then<br />    肯定语句<br />end                      |
| \While{条件}{循环语句}         | while 条件 then<br />    循环语句<br />end                   |
| \tcc{注释}                     | /*注释\*/                                                    |
| \tcp{注释}                     | // 注释                                                      |
| \eIf{条件}{肯定语句}{否定语句} | if 条件 then<br />    肯定语句<br />else<br />    否定语句<br />end |

一个官方的例子如下

```latex
\begin{algorithm}
	\SetKwData{Left}{left}\SetKwData{This}{this}\SetKwData{Up}{up}
	\SetKwFunction{Union}{Union}\SetKwFunction{FindCompress}{FindCompress}
	\SetKwInOut{Input}{input}\SetKwInOut{Output}{output}
	
	\Input{A bitmap $Im$ of size $w\times l$}
	\Output{A partition of the bitmap}
	\BlankLine
	\emph{special treatment of the first line}\;
	\For{$i\leftarrow 2$ \KwTo $l$}{
		\emph{special treatment of the first element of line $i$}\;
		\For{$j\leftarrow 2$ \KwTo $w$}{\label{forins}
			\Left$\leftarrow$ \FindCompress{$Im[i,j-1]$}\;
			\Up$\leftarrow$ \FindCompress{$Im[i-1,]$}\;
			\This$\leftarrow$ \FindCompress{$Im[i,j]$}\;
			\If(\tcp*[h]{O(\Left,\This)==1}){\Left compatible with \This}{\label{lt}
				\lIf{\Left $<$ \This}{\Union{\Left,\This}}
				\lElse{\Union{\This,\Left}}
			}
			\If(\tcp*[f]{O(\Up,\This)==1}){\Up compatible with \This}{\label{ut}
				\lIf{\Up $<$ \This}{\Union{\Up,\This}}
				\tcp{\This is put under \Up to keep tree as flat as possible}\label{cmt}
				\lElse{\Union{\This,\Up}}\tcp*[h]{\This linked to \Up}\label{lelse}
			}
		}
		\lForEach{element $e$ of the line $i$}{\FindCompress{p}}
	}
	\caption{disjoint decomposition}\label{algo_disjdecomp}
\end{algorithm}
```

渲染后的样式如下

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20210118_示例伪代码2.png)

注：当前算法选然后条件语句会带有下划线，主要是因为同时使用了 \ulem 包，产生了冲突，去掉该包即可

当算法过长，超过一页时，由于algorithm2e 没有提供相应的拆分机制，需要自己进行处理，文档描述如下

> Caution: algorithms cannot be cut

自行拆分的方法是，在需要拆开的部分提前加入算法结束符，然后新建算法，示例如下，例子中省略了非必要的代码，但已足够说明如何使用，即只需要新开一个算法块，然后设定行号即可。

```latex
\begin{algorithm}
  $\mathcal{E} \leftarrow \emptyset$\;
\end{algorithm}

\begin{algorithm}
	  \setcounter{AlgoLine}{12}
      $\mathcal{E} \leftarrow \emptyset$\;
      $\mathcal{E} \leftarrow \emptyset$\;
      $\mathcal{E} \leftarrow \emptyset$\;
      $\mathcal{E} \leftarrow \emptyset$\;
      $\mathcal{E} \leftarrow \emptyset$\;     
\end{algorithm}
```



---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2021/pseudocode-specification/  

