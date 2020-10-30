# 提取以太坊的区块产生时间


##  前言

目的是提取以太坊的区块产生时间形成数据集，初步的思路有两种：

- 爬取以太坊区块浏览器中的时间数据，然后格式化形成数据集输出
- 同步以太坊的所有区块（头），利用以太坊本身提供的接口提取每个区块的时间戳信息，然后将时间戳转换为真实的日期格式形成数据集输出，[时间戳转换在线工具](<https://tool.lu/timestamp/>)

查询资料过程中，了解到谷歌已提供以太坊的区块信息数据集[^Bigquery]。虽然数据集本身没有时间信息，但可以导出时间戳进行转换，了解到谷歌使用的工具是[ethereum-etl](<https://github.com/blockchain-etl/ethereum-etl#blockscsv>)，故最终的解决方案是：使用ethereum-etl导出时间戳，编写Python程序转换时间戳并导出数据集。

[^Bigquery]:[谷歌宣布其 BigQuery 服务已支持以太坊区块链数据分析](<https://www.infoq.cn/article/ethereum-bigquery-public-dataset-smart-contract-analytics>)

<!--more-->

## 区块时间戳信息导出

系统环境为Ubuntu18.04，已安装python环境。安装Ethereum ETL：

```bash
$ pip3 install ethereum-etl
```

安装依赖模块

```bash
$ pip3 install mythril
$ pip3 install pyetherchain
```

导出区块信息，导出速度和国家有关，国内较慢。

```bash
$ ethereumetl export_blocks_and_transactions -s 1 -e 200000 -p https://mainnet.infura.io -b 100 -w 3 --blocks-output blocks.csv
```

所有的参数使用可以通过`-h`参数查看，更多信息详见[ethereum-etl](<https://github.com/blockchain-etl/ethereum-etl#blockscsv>)。

```bash
> ethereumetl export_blocks_and_transactions -h

Usage: ethereumetl export_blocks_and_transactions [OPTIONS]

  Export blocks and transactions.

Options:
  -s, --start-block INTEGER   Start block
  -e, --end-block INTEGER     End block  [required]
  -b, --batch-size INTEGER    The number of blocks to export at a time.
  -p, --provider-uri TEXT     The URI of the web3 provider e.g.
                              file://$HOME/Library/Ethereum/geth.ipc or
                              https://mainnet.infura.io
  -w, --max-workers INTEGER   The maximum number of workers.
  --blocks-output TEXT        The output file for blocks. If not provided
                              blocks will not be exported. Use "-" for stdout
  --transactions-output TEXT  The output file for transactions. If not
                              provided transactions will not be exported. Use
                              "-" for stdout
  -h, --help                  Show this message and exit.
```

导出的`blocks.csv`数据集格式如下

| Column            | Type       |
| ----------------- | ---------- |
| number            | bigint     |
| hash              | hex_string |
| parent_hash       | hex_string |
| nonce             | hex_string |
| sha3_uncles       | hex_string |
| logs_bloom        | hex_string |
| transactions_root | hex_string |
| state_root        | hex_string |
| receipts_root     | hex_string |
| miner             | address    |
| difficulty        | numeric    |
| total_difficulty  | numeric    |
| size              | bigint     |
| extra_data        | hex_string |
| gas_limit         | bigint     |
| gas_used          | bigint     |
| timestamp         | bigint     |
| transaction_count | bigint     |

打开`blocks.csv`文件，删除无关项，保留`number`和`timestamp`两项。

## 时间戳转换及数据集生成

利用csv模块进行数据集逐行读取，利用time模块进行时间戳转换，利用numpy模块进行数据集重新写入，代码如下：

```python
import csv,time
import numpy

filename = 'F:/blocks.csv'
with open(filename) as f:
    reader = csv.reader(f)
    header_row = next(reader)
    date = []
    rowNumber = 1
    for row in reader:
        date.append(row)
        rowNumber = rowNumber + 1
    i = 0
    while i < rowNumber-1:
        timeStamp = int(date[i][1])
        timeArray = time.localtime(timeStamp)
        otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S",timeArray)
        date[i].append(otherStyleTime)
        print(date[i][2])
        i = i + 1
    numpy.savetxt('blocks.csv', date, delimiter = ',',fmt = '%s')
```

生成的数据集格式如下，第一列为区块号，第二列为时间戳，第三列为转换后的时间信息，以逗号分隔，共20万条数据。在第一行手动添加表头。

注：此时的以太坊主链总区块数在750万个左右。

```bash
Number,Timestamp,Block generation time
1,1438269988,2015-07-30 23:26:28
2,1438270017,2015-07-30 23:26:57
3,1438270048,2015-07-30 23:27:28
4,1438270077,2015-07-30 23:27:57
5,1438270083,2015-07-30 23:28:03
6,1438270107,2015-07-30 23:28:27
7,1438270110,2015-07-30 23:28:30
8,1438270112,2015-07-30 23:28:32
9,1438270115,2015-07-30 23:28:35
10,1438270128,2015-07-30 23:28:48
11,1438270136,2015-07-30 23:28:56
12,1438270144,2015-07-30 23:29:04
13,1438270158,2015-07-30 23:29:18
14,1438270161,2015-07-30 23:29:21
15,1438270168,2015-07-30 23:29:28
16,1438270174,2015-07-30 23:29:34
```

也可以直接利用谷歌 BigQuery 获取和分析以太坊数据，见[使用谷歌 BigQuery 分析以太坊数据](<https://www.jianshu.com/p/b611dbb526cd>)

## 参考文献

[How to interact with the Ethereum blockchain and create a database with Python and SQL](<https://medium.com/validitylabs/how-to-interact-with-the-ethereum-blockchain-and-create-a-database-with-python-and-sql-3dcbd579b3c0>)

[How do you work with Date and time on Ethereum platform](<https://ethereum.stackovernet.com/cn/q/5558>)

[Google Cloud-Ethereum in BigQuery](<https://cloud.google.com/blog/products/data-analytics/ethereum-bigquery-public-dataset-smart-contract-analytics>)

[Google Cloud数据库操作](<https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types>)

[Google Cloud文档](<https://googleapis.github.io/google-cloud-python/latest/bigquery/usage/queries.html>)

[How to use Kaggle](<https://www.kaggle.com/docs/datasets>)

[Kaggle调用BigQuery](<https://www.kaggle.com/bigquery/ethereum-blockchain>)

[Kaggle-Beyond Queries: Exploring the BigQuery API](<https://www.kaggle.com/sohier/beyond-queries-exploring-the-bigquery-api>)

[Kaggle-Visualizing average Ether costs over time](<https://www.kaggle.com/mrisdal/visualizing-average-ether-costs-over-time/data>)
