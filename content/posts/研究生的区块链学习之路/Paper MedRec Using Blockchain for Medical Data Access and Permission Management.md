---
title: MedRec:Using Blockchain for Medical Data Access and Permission Management
date: 2019-01-18T16:37:00+08:00
tags: [论文笔记]
categories: [研究生的区块链学习之路]
---

Author：Asaph Azaria ; Ariel Ekblaw ; Thiago Vieira ; Andrew Lippman

Published in：2016 2nd International Conference on Open and Big Data (OBD)

被引数：217次

这篇论文只是一份白皮书的一部分，白皮书全文见：

[A Case Study for Blockchain in Healthcare: “MedRec” prototype for electronic health records and medical research data](https://pdfs.semanticscholar.org/56e6/5b469cad2f3ebd560b3a10e7346780f4ab0a.pdf)

### 摘要

电子病历（EMRs）发展停滞，需要创新。本文提出了MedRec，一种区块链技术管理EMR的系统，为病患提供全面的不可变的病历记录和对其病历的轻松访问。基于区块链的特殊属性，它能提供身份验证，机密性，问责机制，和处理敏感信息时考虑的关键因素—数据共享。模块化设计和供应商现有的本地数据存储方案相集成，使系统更方便，适应性更高。我们鼓励医药业相关人员（如研究人员和卫生局）作为矿工参与该网络，采用PoW挖矿的奖励是对数据的访问权限。所以可以吸引研究人员的参与。

<!--more-->

### I. 引言

患者的病历由各医院管理，而各医院间没有病历的统一管理，所以病历通常是一个个的碎片，患者无法获得和访问一个完整的病历记录。这种情况会增加患者就医的麻烦程度，也不利于医生给出确切的诊断，甚至有可能被某些人用以牟利。另外，病历对研究也至关重要，医学研究人员往往需要这些记录来识别公共健康风险和开发新的治疗方法。

本文中，我们探索将区块链结构用于EMR，最初建立在比特币的不可变分类账上。这种结合解决了四个主要的问题

- 对医疗数据碎片化地，缓慢地访问
- 各医院系统的互操作性
- 病人代理
- 提高了用于医疗研究的数据质量和数量

通过聚集不同医疗数据的引用并把它们编码到区块链，提供了基于区块链的访问许可和数据完整性记录，赋予了医疗记录以真实性、可审计性和数据共享的权力。构建了模块化API和现有的医疗数据库集成以实现互操作，并提出一种新的共识方式来维持MedRec网络并未研究人员提供大数据。

### II. 现有技术

实际上是相关工作，介绍了几个相似的工作

### III. 系统实施

#### A. Overview

