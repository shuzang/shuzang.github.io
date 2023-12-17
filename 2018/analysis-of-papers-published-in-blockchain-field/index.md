# 背景调查2-近三年区块链方向论文发表情况总结


原始的目的是统计一下国内区块链研究现状，但开始这一工作时决定顺便把其它的统计数据也一并记录下来。数据来自

- [Web of Science核心合集]( http://apps.webofknowledge.com/WOS_GeneralSearch_input.do?product=WOS&search_mode=GeneralSearch&SID=6CIWceJqI9n1qQa94CZ&preferencesSaved= )，即SCI，以`blockchain`为关键词，总计3399篇论文
- [EI Compendex]( https://www.engineeringvillage.com/search/quick.url?SEARCHID=30ec8294fa0e452ba59525594e492cda&COUNT=1&usageOrigin=header&usageZone=evlogo#foo )，即EI，以`blockchain为`关键词，总计6659篇论文
- CNKI核心期刊，会议，硕博论文

## 1. 学科

论文所属学科以计算机为主，SCI中工程电子电气、计算机科学理论方法、计算机科学信息系统、电信四个学科是最主要的发表领域，剩下的论文数量超过100篇的学科分别是计算机科学的跨学科应用、计算机科学软件工程、计算机科学硬件体系结构、计算机科学人工智能和能源燃料。随后的两个领域分别是工业工程和自动化控制系统，而直观上感觉相关的经济学只排到第13位。

SCI论文所属学科论文统计的图片丢失

EI中结果仿佛，柱状图如下所示

![EI论文所属学科论文统计](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181117_EI论文所属学科统计.png)

## 2. 出版年

虽然比特币起始自2008年，但区块链的论文直到2013年才有收录，并且只有2篇，2018、2019两年达到最高，每年都有近1400篇。柱状图如下

（图片丢失）

具体每年的数据如下表所示，有趣的是今天才2019年12月2日，结果2020年待发表论文已经有23篇了，超过刚发展的前三年。

| 出版年 | 论文数 | 百分比 |
| ------ | ------ | ------ |
| 2020   | 23     | 0.677  |
| 2019   | 1385   | 40.747 |
| 2018   | 1392   | 40.953 |
| 2017   | 466    | 13.710 |
| 2016   | 99     | 2.913  |
| 2015   | 22     | 0.647  |
| 2014   | 10     | 0.294  |
| 2013   | 2      | 0.059  |

EI论文总体的发表趋势相同，都是2017年开始有明显增长，从2018年开始爆发

![发表年份](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181117_EI论文发表年份统计.png)

两张图中都是在大概2013年起才出现论文，这可能是因为之前`区块链`这一概念并没有得到重视，相关的论文关键字很可能是`比特币`

## 3. 文献类型

SCI中会议论文2136篇，占比62.842%；期刊论文1065篇，占比31.333%；综述86篇，占比2.530%。

文献类型统计图片丢失

其中会议论文一共2136篇，分散在953个会议中。发表区块链论文最多的两个会议是 IEEE International Congress on Cybermatics 和 IEEE International Conference on Blockchain and Cryptocurrency，后者在2019年第一次举办，是IEEE举办的第一个区块链和加密货币的高级会议。下面是发表论文在10篇及以上的会议名。

| 会议名                                                       | 数量 | 百分比  |
| :----------------------------------------------------------- | :--- | :------ |
| IEEE International Congress on Cybermatics                   | 97   | 2.854 % |
| 1ST IEEE International Conference on Blocakchain and Cryptocurrency(ICBC) | 74   | 2.177 % |
| IEEE International Conference on Communications(ICC)         | 33   | 0.971 % |
| 1ST IEEE International Conference on Hot Information-Centric Networking(HotICN) | 26   | 0.765 % |
| 18TH IEEE International Conference on Data Mining workshops(ICDMW) | 15   | 0.441 % |
| 24TH IEEE International Conference on Papallel and Distributed Systems(ICPADS) | 14   | 0.412 % |
| 9TH IFIP International Conference on New Technologies Mobility and Security(NTMS) | 14   | 0.412 % |
| Crypto Valley Conference on Blockchain Technology(CVCBT)     | 14   | 0.412 % |
| 11TH IEEE International Conference on Cloud Computing, Cloud Part of The IEEE World Congress on Services | 13   | 0.382 % |
| 34TH ACM Sigapp Annual International Symposium on Applied Computing(SAC) | 13   | 0.382 % |
| 4TH International Conference on Cloud Computing and Security(ICCCS) | 13   | 0.382 % |
| IEEE Global Telecommunications Conference GC(WKSHPS)         | 13   | 0.382 % |
| 10TH IFIP International Conference on New Technologies Mobility and Security(NTMS) | 12   | 0.353 % |
| 21ST International Conference on Advanced Communication Technology(ICACT) | 12   | 0.353 % |
| IEEE International Conference on Big Data                    | 12   | 0.353 % |
| International Conference on Blockchain Technology(ICBCT)     | 12   | 0.353 % |
| 5TH IEEE World Forum on Internet of Things(IEEE WF IOT)      | 11   | 0.324 % |
| 1ST ACM IEEE International Workshop on Emerging Trends in Software Engineering for Blockchain(WETSEB) | 10   | 0.294 % |
| 2ND International Workshop on Cryptocurrencies and Blockchain Technology(CBT) <br>13TH International Workshop on Data Privacy Management(DPM) | 10   | 0.294 % |
| IEEE Global Communications Conference                        | 10   | 0.294 % |
| International Conference on Computing Networking and Communications(ICNC) | 10   | 0.294 % |

期刊最多的是IEEE Access，183篇，IEEE Internet of Things排第五，44篇，Sensors排第七，34篇，IEEE Transactions on Industrial Informatics排十一，25篇。更多的数据见下表，只统计20篇及以上的，虽然不知道为什么有些会议也算进去了

| 期刊名                                                       | 数量 | 百分比  |
| :----------------------------------------------------------- | :--- | :------ |
| IEEE ACCESS                                                  | 183  | 5.384 % |
| Lecture Notes in Computer Science                            | 121  | 3.560 % |
| 2018 IEEE International Congress on Cybermatics              | 97   | 2.854 % |
| 2019 IEEE International Conference on Blockchain and Cryptocurrency(ICBC) | 74   | 2.177 % |
| IEEE Internet of Things Journal                              | 44   | 1.294 % |
| Advances in Intelligent Systems and Computing                | 37   | 1.089 % |
| Sensors                                                      | 34   | 1.000 % |
| IEEE International Conference on Communications              | 27   | 0.794 % |
| International Conference on New Technologies Mobility and Security | 26   | 0.765 % |
| Proceedings of 20181ST IEEE International Conference on Hot Information Centric Networking(HotICN 2018) | 26   | 0.765 % |
| IEEE Transactions on Industrial Informatics                  | 25   | 0.736 % |
| Applied Sciences Basel                                       | 23   | 0.677 % |
| 2019 IEEE International Conference on Communications(ICC)    | 23   | 0.677 % |
| Future Generation Computer Systems The International Journal of Escience | 22   | 0.647 % |
| Sustainability                                               | 22   | 0.647 % |
| International Conference on Parallel and Distributed Systems Proceedings | 20   | 0.588 % |
| IT Professional                                              | 20   | 0.588 % |

EI中文献类型占比如下图，会议论文4612篇，期刊论文1290篇，会议论文集532，报刊170，图书章节29，图书6，勘误表3篇，社论1篇。

![文献类型](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181117_EI论文文献类型统计.png)

具体的文献来源如下表所示，代表了发表收录区块链领域论文数量最多的会议和期刊

| Source title                                                 | Count |
| ------------------------------------------------------------ | ----- |
| Lecture Notes In Computer Science  (Including Subseries Lecture Notes In Artificial Intelligence And Lecture  Notes In Bioinformatics) | 662   |
| Proceedings - Ieee 2018 International  Congress On Cybermatics | 306   |
| Acm International Conference Proceeding  Series              | 245   |
| Advances In Intelligent Systems And  Computing               | 183   |
| Ieee Access                                                  | 160   |
| Ceur Workshop Proceedings                                    | 111   |
| Communications In Computer And  Information Science          | 104   |
| Lecture Notes In Business Information  Processing            | 100   |
| Icbc 2019 - Ieee International  Conference On Blockchain And Cryptocurrency | 84    |
| Future Generation Computer Systems                           | 63    |
| Proceedings Of 2018 1st Ieee  International Conference On Hot Information-Centric Networking, Hoticn 2018 | 58    |
| Lecture Notes Of The Institute For  Computer Sciences, Social-Informatics And Telecommunications Engineering,  Lnicst | 56    |
| Ieee Internet Of Things Journal                              | 41    |
| Ieee International Conference On  Communications             | 38    |
| International Conference On Cloud  Computing, Big Data And Blockchain, Iccbb 2018 | 37    |
| Sensors (Switzerland)                                        | 37    |
| Sensors                                                      | 37    |
| Journal Of Physics: Conference Series                        | 34    |
| Proceedings - International Conference  On Distributed Computing Systems | 33    |
| Proceedings Of The Acm Conference On  Computer And Communications Security | 32    |
| Lecture Notes In Electrical Engineering                      | 29    |
| Iop Conference Series: Materials Science  And Engineering    | 29    |
| Procedia Computer Science                                    | 26    |
| Ifip Advances In Information And  Communication Technology   | 25    |
| It Professional                                              | 25    |
| Proceedings - International Conference  On Software Engineering | 25    |
| Ieee Transactions On Industrial  Informatics                 | 24    |
| Ruan Jian Xue Bao/Journal Of Software                        | 23    |
| Energies                                                     | 22    |
| Proceedings Of The International  Conference On Parallel And Distributed Systems - Icpads | 22    |
| Proceedings Of Spie - The International  Society For Optical Engineering | 22    |
| Handbook Of Blockchain, Digital Finance,  And Inclusion, Volume 1: Cryptocurrency, Fintech, Insurtech, And Regulation | 22    |
| Cryblock 2018 - Proceedings Of The 1st  Workshop On Cryptocurrencies And Blockchains For Distributed Systems, Part Of  Mobisys 2018 | 21    |
| Leibniz International Proceedings In  Informatics, Lipics    | 21    |
| Proceedings - International Computer  Software And Applications Conference | 21    |
| Proceedings - 2019 Ieee International  Conference On Decentralized Applications And Infrastructures, Dappcon 2019 | 21    |
| Ieee International Conference On Cloud  Computing, Cloud     | 19    |
| Smart Innovation, Systems And  Technologies                  | 19    |
| Proceedings - 17th Ieee International  Conference On Trust, Security And Privacy In Computing And Communications And  12th Ieee International Conference On Big Data Science And Engineering,  Trustcom/Bigdatase 2018 | 18    |
| Jisuanji Yanjiu Yu Fazhan/Computer  Research And Development | 18    |
| Infocom 2019 - Ieee Conference On  Computer Communications Workshops, Infocom Wkshps 2019 | 18    |
| International Journal Of Information  Management             | 17    |
| International Conference On Information  Systems 2018, Icis 2018 | 17    |
| Bsci 2019 - Proceedings Of The 2019 Acm  International Symposium On Blockchain And Secure Critical Infrastructure,  Co-Located With Asiaccs 2019 | 17    |
| Computers And Security                                       | 17    |
| Future Internet                                              | 17    |
| Proceedings - 2018 Crypto Valley  Conference On Blockchain Technology, Cvcbt 2018 | 17    |
| Iop Conference Series: Earth And  Environmental Science      | 17    |
| Matec Web Of Conferences                                     | 16    |
| International Conference On Advanced  Communication Technology, Icact | 16    |
| Ieee Transactions On Computational  Social Systems           | 16    |
| Ieee International Conference On Data  Mining Workshops, Icdmw | 16    |
| Americas Conference On Information  Systems 2018: Digital Disruption, Amcis 2018 | 16    |
| Zidonghua Xuebao/Acta Automatica Sinica                      | 16    |
| Proceedings Of The Acm Symposium On  Applied Computing       | 16    |
| Zhongguo Dianji Gongcheng  Xuebao/Proceedings Of The Chinese Society Of Electrical Engineering | 15    |
| Concurrency Computation                                      | 15    |
| Information Sciences                                         | 15    |
| 25th Americas Conference On Information  Systems, Amcis 2019 | 15    |
| Concurrency Computation Practice And  Experience             | 15    |

CCF在2018年4月成立了[区块链专业委员会](https://www.ccf.org.cn/tcbc/)，组织的代表性活动有中国区块链技术与应用高峰论坛和中国区块链技术大会，后者更影响力更大。

[中国区块链技术和产业发展论坛](http://www.cbdforum.cn/bcweb/)则是中国电子技术标准化研究院发起的，致力于中国区块链标准化体系建设，每年发表一份中国区块链技术和应用发展研究报告。

从CNKI收录区块链的期刊检索的结果中寻找，选取影响因子大于1的

- [电子与信息学报](http://navi.cnki.net/KNavi/JournalDetail?pcode=CJFD&pykm=DZYX&Year=&Issue=&Entry=&uid=WEEvREcwSlJHSldRa1FhcTdWa2FjcWpzRm0yUUJ0MldDY0QxcEJCY1RVcz0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!), EI
- [计算机研究与发展](http://navi.cnki.net/KNavi/JournalDetail?pcode=CJFD&pykm=JFYZ&Year=&Issue=&Entry=&uid=WEEvREcwSlJHSldRa1FhcTdWa2FjcWpzRm0yUUJ0MldDY0QxcEJCY1RVcz0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!), EI
- 中国科学：信息科学
- [软件学报](http://navi.cnki.net/KNavi/JournalDetail?pcode=CJFD&pykm=RJXB&Year=&Issue=&Entry=&uid=WEEvREcwSlJHSldRa1FhcTdWa2FjcWpzRm0yUUJ0MldDY0QxcEJCY1RVcz0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!)，EI
- [计算机应用研究](http://navi.cnki.net/KNavi/JournalDetail?pcode=CJFD&pykm=JSYJ&Year=&Issue=&Entry=&uid=WEEvREcwSlJHSldRa1FhcTdWa2FjcWpzRm0yUUJ0MldDY0QxcEJCY1RVcz0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!)
- [通信学报](http://navi.cnki.net/KNavi/JournalDetail?pcode=CJFD&pykm=TXXB&Year=&Issue=&Entry=&uid=WEEvREcwSlJHSldRa1FhcTdWa2FjcWpzRm0yUUJ0MldDY0QxcEJCY1RVcz0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!)，EI
- [电子学报](http://navi.cnki.net/KNavi/JournalDetail?pcode=CJFD&pykm=DZXU&Year=&Issue=&Entry=&uid=WEEvREcwSlJHSldRa1FhcTdWa2FjcWpzRm0yUUJ0MldDY0QxcEJCY1RVcz0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!)，EI
- 信息网络安全
- [计算机学报](http://navi.cnki.net/KNavi/JournalDetail?pcode=CJFD&pykm=JSJX&Year=&Issue=&Entry=&uid=WEEvREcwSlJHSldRa1FhcTdWa2FjcWpzRm0yUUJ0MldDY0QxcEJCY1RVcz0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!)，EI

## 4. 机构

首先是国家归属，中美最多，英国、印度、德国、韩国、澳大利亚、加拿大、意大利、法国等紧随其后，下图上半部分是SCI的论文统计柱状图，下图下半部分是EI的统计，两者的结果几无差别。

论文归属国统计图片丢失

基金资助的机构显然与国家相关，前两位是中国的国家自然科学基金和美国国家科学基金。

基金资助机构SCI排序统计图片丢失

![基金资助机构EI排序](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181117_EI论文基金资助机构统计.png)

在出版商中，SCI前两位是IEEE、ACM，而EI中则是IEEE、Springer，ACM排第三，不过两个数据库中IEEE都占最大的比重。

## 5. 小型组织

小型的组织的含义是各种大学、研究院、公司等，SCI中占比最多的是北京邮电大学和中科院，都在50篇以上。详细数据列表如下，统计以20篇为下限

| 组织                           | 论文数 |
| ------------------------------ | ------ |
| 北京邮电大学                   | 75     |
| 中科院                         | 63     |
| 伦敦大学                       | 47     |
| IBM                            | 46     |
| 澳大利亚联邦科学与工业研究组织 | 43     |
| 中国电子科技大学               | 39     |
| 澳大利亚新南威尔士大学悉尼校区 | 38     |
| 德克萨斯大学系统               | 38     |
| 北京航空航天大学               | 36     |
| 新加坡南洋理工大学             | 36     |
| 加州大学系统                   | 36     |
| 清华大学                       | 34     |
| 宾夕法尼亚联邦高等教育系统     | 32     |
| 西安电子科技大学               | 32     |
| 上海交通大学                   | 29     |
| 德克萨斯大学圣安东尼奥分校     | 27     |
| 中国国防科技大学               | 26     |
| 北京大学                       | 26     |
| 悉尼大学                       | 26     |
| 印度理工学院                   | 25     |
| 佛罗里达州立大学系统           | 25     |
| 中山大学                       | 25     |
| 乔治亚大学系统                 | 25     |
| 广东工业大学                   | 24     |
| 北京理工大学                   | 23     |
| 法国国家科学研究中心           | 23     |
| 新加坡国立大学                 | 23     |
| 瑞士苏黎世联邦理工学院         | 22     |
| 阿联酋哈利法科学技术大学       | 22     |
| 意大利卡利亚里大学             | 22     |
| 坦普尔大学                     | 21     |
| 悉尼科技大学                   | 21     |
| 麻省理工学院                   | 20     |
| 美国国防部                     | 20     |
| 加拿大英属哥伦比亚大学         | 20     |
| 中国科学院大学                 | 20     |
| 卢森堡大学                     | 20     |
| 北卡罗莱纳大学                 | 20     |

德克萨斯大学系统的38中有27篇都是圣安东尼奥分校贡献的；新加坡南洋理工大学和新加坡南洋理工大学国立教育学院的36篇是重合的，因此只记录了一个；伦敦大学47篇有22篇是伦敦大学学院贡献的，因此伦敦大学学院没有记录在上面；宾夕法尼亚州的Temple University译名不统一，上表中记录为坦普尔大学；英属哥伦比亚大学有时也称不列颠哥伦比亚大学，上表记为前者。

将上表中中国的研究机构单独抽离为下表，共13个机构，

| 组织             | 论文数 |
| ---------------- | ------ |
| 北京邮电大学     | 75     |
| 中科院           | 63     |
| 中国电子科技大学 | 39     |
| 北京航空航天大学 | 36     |
| 清华大学         | 34     |
| 西安电子科技大学 | 32     |
| 上海交通大学     | 29     |
| 中国国防科技大学 | 26     |
| 北京大学         | 26     |
| 中山大学         | 25     |
| 广东工业大学     | 24     |
| 北京理工大学     | 23     |
| 中国科学院大学   | 20     |

北京邮电大学高被引论文两篇，研究领域分别是能量网络和车联网，以第一作者发表论文较多的有：Liu Mengting 6篇，北京天地互联与融合重点实验室, Yang Hui 4篇光子光与光通信重点实验室，马兆丰 4篇 区块链及安全技术联合实验室

中科院高被引论文1篇，领域为智能交通系统，作者是Yuan Yong，该作者发表的另一篇引用数也在17。除此之外，Fan Kai以一作身份发表四篇，

东北大学有[五篇](http://apps.webofknowledge.com/Search.do?product=WOS&SID=8AwkL1E67zOjrP8lBCU&search_mode=GeneralSearch&prID=d81b18ad-488c-4ec2-950e-4dfffbce18a6)，第一作者有鲁宁副教授(分校)和刘园副教授(软件学院)，剩下两个第一作者教师名单查不到。

EI中按组织对论文数量排序如下表，可能统计规则有所区别，EI的数据明显少很多，单个机构最多也只有37篇，因此以10篇为下限。

| Author affiliation                       | Count |
| ---------------------------------------- | ----- |
| Ibm                                      | 37    |
| 澳大利亚联邦科学与工业研究组织           | 33    |
| 瑞士苏黎世联邦理工学院                   | 27    |
| 青岛智能产业技术研究院                   | 21    |
| 北京邮电大学网络与交换技术国家重点实验室 | 20    |
| 伦敦大学学院                             | 18    |
| 中国科学院大学                           | 18    |
| 南洋理工大学                             | 17    |
| 卢森堡大学                               | 17    |
| 萨斯喀彻温大学                           | 16    |
| 中山大学                                 | 15    |
| 中国电子科技大学                         | 15    |
| 新加坡国立大学                           | 15    |
| 慕尼黑工业大学                           | 15    |
| 西安电子科技大学                         | 14    |
| 山东大学                                 | 14    |
| 坦普尔大学                               | 14    |
| 普渡大学                                 | 14    |
| 中科院                                   | 14    |
| 新南威尔士大学                           | 13    |
| 加利福尼亚大学                           | 13    |
| 卡尔顿大学                               | 13    |
| 中佛罗里达大学                           | 13    |
| 亚利桑那州立大学                         | 13    |
| 北京工业大学                             | 13    |
| 伊利诺伊大学香槟分校                     | 12    |
| 休斯顿大学                               | 12    |
| 香港理工大学                             | 12    |
| 斯坦福大学                               | 12    |
| 意大利卡利亚里大学                       | 12    |
| 英国南安普顿大学                         | 12    |
| 澳大利亚莫纳什大学                       | 11    |
| 塔尔图大学                               | 10    |
| 广州大学                                 | 10    |
| 荷兰代尔夫特理工大学                     | 10    |
| 德克萨斯大学圣安东尼奥分校               | 10    |
| 南京信息工程大学                         | 10    |
| 康奈尔大学                               | 10    |
| 欧道明大学弗吉尼亚建模分析与仿真中心     | 10    |
| 新加坡管理大学                           | 10    |

依然单独抽离中国的组织

| Author affiliation                       | Count |
| ---------------------------------------- | ----- |
| 青岛智能产业技术研究院                   | 21    |
| 北京邮电大学网络与交换技术国家重点实验室 | 20    |
| 中国科学院大学                           | 18    |
| 中山大学                                 | 15    |
| 中国电子科技大学                         | 15    |
| 西安电子科技大学                         | 14    |
| 山东大学                                 | 14    |
| 中科院                                   | 14    |
| 北京工业大学                             | 13    |
| 香港理工大学                             | 12    |
| 广州大学                                 | 10    |
| 南京信息工程大学                         | 10    |

北邮、中科院、电子科大、西电依然在表中，但其它的大学则和SCI的统计数据略微不同。东北大学有[八篇](https://www.engineeringvillage.com/search/quick.url?SEARCHID=710b220ca03e40c38f443042355b5765&COUNT=1&usageOrigin=header&usageZone=evlogo)，其中四篇和SCI收录重合

国内CNKI数据的检索结果中，核心期刊被引量排名前200篇论文的作者与所属组织分布如下

![期刊作者与机构分布](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181117_CNKI期刊作者与机构分布.png)

会议如下

![会议作者与机构分布](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181117_CNKI会议作者与机构分布.png)

硕博论文如下，以区块链方向论文毕业的学生，专业有一半以上都位于计算机和通信。该领域毕业论文数量较多的学校如下图，这在一定程度上反映了该学校在此领域的研究热度，北京邮电不出意外是最多的。

![硕博论文所属大学排行](https://picped-1301226557.cos.ap-beijing.myqcloud.com/70035204-668a6600-15ed-11ea-8034-c432b06dbd23.png)

## 6. 作者

以作者来看的话，SCI发表情况如下所示

SCI作者统计图片丢失

不过这种统计看起来有问题，比如Zhang Y，所有姓名中第二个字拼音首字母为Y的人都被统计进来了，不是一个人。因此换一种方法，将条件限制到领域中的高被引论文和热点论文，以及排在前五位的中国基金，查看第一作者的情况，最后得到15篇。这些作者包括

- 广东工业大学，Kang Jiawen，2篇
- 华南理工大学，Xia Qi
- 湘潭大学，Li Zhetao, [Consortium Blockchain for Secure Energy Trading in Industrial Internet of Things](http://apps.webofknowledge.com/full_record.do?product=WOS&search_mode=GeneralSearch&qid=107&SID=8AwkL1E67zOjrP8lBCU&page=1&doc=3&cacheurlFromRightClick=no)
- 中山大学，Zheng Zibin,  [Blockchain challenges and opportunities: a survey](http://apps.webofknowledge.com/full_record.do?product=WOS&search_mode=GeneralSearch&qid=107&SID=8AwkL1E67zOjrP8lBCU&page=1&doc=4&cacheurlFromRightClick=no)
- 中山大学，Liu Hong
- 韩山师范，Lin Qun
- 丹麦技术大学，Meng Weizhi,  [When Intrusion Detection Meets Blockchain Technology: A Review](http://apps.webofknowledge.com/full_record.do?product=WOS&search_mode=GeneralSearch&qid=107&SID=8AwkL1E67zOjrP8lBCU&page=1&doc=6&cacheurlFromRightClick=no)
- 华北电力大学， Guan Zhitao
- 西安邮电大学，Zhang Yinghui
- 西安邮电大学，Guo Rui
- 上海大学，Su Zhou
- 奈良先端科学技术大学院大学，Zhang Yuanyu， [Smart Contract-Based Access Control for the Internet of Things](http://apps.webofknowledge.com/full_record.do?product=WOS&search_mode=GeneralSearch&qid=107&SID=8AwkL1E67zOjrP8lBCU&page=2&doc=13&cacheurlFromRightClick=no)
- 中国电子科技大学，Chen Yi
- 北京理工大学，Gai Keke

广东工业的Kang Jiawen一个人发了两篇高被引，除此之外，中山大学两篇，西安邮电两篇。

EI的统计则是以全名为依据，重复概率较小，以柱状图来看，前十位中，王飞跃 和 袁勇都是中科院自动化所，后者第一作者身份发表的区块链文章更多，Xu Xiwei和Weber Ingo属于澳大利亚联邦科学与工业研究组织，Xu Lei为休斯顿大学，Lei Kai属于北京大学深圳信息中心网络与区块链技术重点实验室，Niyato Dusit南洋理工大学，Norta Alex爱沙尼亚塔林理工大学，Du Xiaojiang和Choo Kim-Kwang Raymond都没有发现一作论文。

![作者](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181117_EI论文作者统计.png)

不晓得国外的情况怎么样，国人名字为主的论文中，挂名情况严重，导致统计结果很不理想，一个人名下可能几十篇论文，但全是非第一作者，甚至全是后面无关紧要的作者，毫无参考价值。

**课题组：**

- [东北大学区块链实验室](http://blockchain-neu.com/)

## 7. 总结

总体就是这么个情况了，下面稍微做一些总结，列举我们可得到的一些结论

- 论文发表领域以计算机和通信为主
- 区块链领域的论文数量自2017年开始快速增长，2018年爆发，2019年与2018年持平，预计2020年将持平或更多，说明区块链领域依然火热
- 中美是区块链领域成果发表的主要国家
- 北邮、中科院、电子科大、北航、清华、西电、上交、国防科大、北大、中山、广东工业、北京理工是中国区块链的主要阵地，中山和广东工业在高被引论文方面位于前列
- 作者中，中科院自动化所的袁勇成果最为突出


---

> 作者:   
> URL: https://shuzang.github.io/2018/analysis-of-papers-published-in-blockchain-field/  

