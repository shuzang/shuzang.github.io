# ISBN说明


开启随机标准书号生成项目，首先了解ISBN。

国际标准书号（International Standard Book Number），简称ISBN，是专门为识别图书等文献而设计的国际编号。ISO于1972年颁布了ISBN国际标准，并在西柏林普鲁士图书馆设立了实施该标准的管理机构——国际ISBN中心。2007年1月1日起，启用新版ISBN，由13位数字组成，以四个连接号`-`或空格分割为五部分，分别是GSI前缀、组区号、出版者号、出版序号和校验码，首尾两部分长度固定，中间的组区号、出版者号、出版序号位数是变化的。具体可参考[ISBN用户手册中文版]([https://www.isbn-international.org/sites/default/files/ISBN%20users%27%20Manual%202017-simplified%20chinese%20%28Chinese%20translation%20of%20seventh%20edition%29.pdf](https://www.isbn-international.org/sites/default/files/ISBN users' Manual 2017-simplified chinese (Chinese translation of seventh edition).pdf)。

![example](https://www.isbn-international.org/sites/default/files/styles/internal_page/public/what%20is%20an%20ISBN%20%281%29%20English%20revised.png?itok=FdECBknf)

> ISBN官方网站为：[https://www.isbn-international.org/](https://www.isbn-international.org/)

## 各字段解释

### 1. GSI前缀

ISBN 的第一部分是 GS1 前缀，由 GS1（国际物品编码协会，原名EAN） 提供三位数字，当前为 978 和 979。为了确保 ISBN 系统的持续编码能力，将来会做进一步的前缀分配。

### 2. 组区号

ISBN 的第二部分是组区号，是 ISBN 系统所在的国家、地区或语言地区。部分 ISBN 系统成员来自语言地区（例如，组区号 3=德语区）；其他区域地区（例如，组区号 982=南太平洋）。这部分的数字长度是变化的，最多由 5 位数字组成。组区号由国际 ISBN 中心分配。

常用组区号列表如下(2019.10，来自[International ISBN Agency - Range Message-pdf sorted by prefix](https://www.isbn-international.org/range_file_generation))：

| 国家、地区或语言地区   | 缩写 | 组区号             |
| ---------------------- | ---- | ------------------ |
| English language       | en   | 978-0<br>978-1     |
| French language        | fr   | 978-2              |
| German language        | de   | 978-3              |
| Japan                  | jp   | 978-4              |
| former U.S.S.R(Russia) | ru   | 978-5              |
| China                  | cn   | 978-7              |
| Taiwan                 | tw   | 978-957<br>978-986 |
| Hong Kong              | hk   | 978-962<br>978-988 |
| France                 | fr   | 979-10             |

其实只收录英语和中文就行了，不过组区号前十位的都是蛮常用的国家，就一起记录了

### 3. 出版者号

ISBN 的第三部分是出版者号，即出版社代码，是出版社或注册组区内的出版印记。这部分的数字长度是变化的，直接关系到出版者的计划出版量，最多由 7 位数字组成。出版量最大的出版者获得的出版者号长度最短，反之亦然。
出版社向其所在地负责的国家、地区或语言地区 ISBN 系统管理的 ISBN 组区管理机构申请唯一的出版者号，一旦用尽与其出版者号相关联的 ISBN 号，可以再申请新的出版者号。

出版者号不是按数字连续排列的，中国的出版者号范围如下，可从[官网](https://www.isbn-international.org/agencies)查阅

| prefix: 978-7 |
| ------------- |
| 00-99         |
| 100-499       |
| 5000-7999     |
| 80000-89999   |
| 900000-999999 |

实际上，这些序号并没有排满，目前，01是人民出版社，02是人民文学出版社，03是科学出版社，04是高等教育出版社，再往后就是100（商务出版社了）。具体的出版社和代码的对应参见[中国(大陆)出版社ISBN代码表](https://www.douban.com/group/topic/3833361/)

目前的范围为：01-04，100-121，20-228，300-314，5000-5092，5300-5447，5600-5641，80000-80227，80500-80733，81000-81115

### 4. 出版序号

ISBN 的第四部分是出版序号，代表一个具体出版者的某出版物的特定版次。这部分的数字长度是变化的，直接关系到出版者的计划出版量，最多由 6 位数字组成。最大预期出版量的出版者得到的书名号长度最长，反之亦然。为了确保维持 ISBN 号的正确长度，一般使用 0 来代替前面的空格。

### 5. 校验位

ISBN 的第五部分是校验位。将前 12 位数字每一个都交替乘以 1 和 3。然后以模数 10 减去前 12 位数加权乘积之和除以 10 的余数，所得即为校验位。有一个例外，如果这个计算结果显示为 10，则校验位为 0。
以计算 ISBN 978-92-95055-12-？的校验位为例。

1. 确定该 ISBN 号前 12 位数加权乘积之和（见下表）

    ![ISBN校验位计算](/images/ISBN说明/1HtOBT.png)

2. 将步骤 1 中计算的前 12 位数的加权乘积的和除以 10，确定余数

    126 / 10 = 12，余数为6

3. 用 10 中减去步骤 2 计算的余数。所得即校验位。如果步骤 2 的余数为 0，则校验位为 0

    10 - 6 = 4，校验位为4

因此，ISBN = 978-92-95055-12-4

## 适用范围

在中国，ISBN的适用范围包括

（1）印刷的图书和小册子（以及此类出版物的不同产品形式）；
（2）盲文出版物；
（3）出版者无计划定期更新或无限期延续的出版物；
（4）教育或教学用影片、录像制品和幻灯片；
（5）磁带和CD或DVD形式的有声读物；
（6）电子出版物实物载体形式（机读磁带、光盘、CD-ROMs）或是在互联网上出版的电子出版物；
（7）印刷出版物的电子版；
（8）缩微出版物；
（9）教育或教学软件；
（10）混合媒体出版物（内容以文字材料为主的）；
（11）地图及教学制图、图示类出版物。

因此，不只包含图书，在写代码的时候应注意只搜索图书。

## 文献

中国ISBN中心：https://yiliudaxue.com/zhi/v/430045/

ISBN 官方：https://www.isbn-international.org/agencies

中国出版社ISBN代码：https://blog.csdn.net/mighty13/article/details/78088570

中国版本图书馆：https://www.capub.cn/shtmsl/index_shtmsl.shtml?id=8

[百度百科-国际标准书号](https://baike.baidu.com/item/国际标准书号/3271472?fromtitle=ISBN&fromid=391662)

ISBN搜索：https://isbnsearch.org/isbn/9784561107828

github ISBN生成项目：https://github.com/hidapple/isbn-gen
