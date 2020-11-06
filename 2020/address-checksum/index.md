# Address checksum


在智能合约中显式传入地址类型时，可能会出现如下错误

> Address checksum
>
> This looks like an address but has an invalid checksum. If this is not used as an address, please prepend '00'. 

关于该问题的一个讨论见 https://github.com/ethereum/EIPs/issues/55 

这是因为合约中现在使用地址类型必须做一个转换，不是简单的全部大写字母或小写字母，而是遵循一定的规则，这个规则见 [ethereum/EIPs#55]( https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md ) 

但是网上提供的解决方案一般是使用JS库中的转换函数，在智能合约中无法直接解决，好在，web3提供了一个[在线API接口](https://web3-tools.netlify.com/)，可以调用其`checkAddressChecksum`函数对地址进行转换，然后将转换后的结果直接用于合约代码。
