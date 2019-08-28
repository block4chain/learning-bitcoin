<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**目录**

- [名词解释](#名词解释)
    - [不定长整型](#不定长整型)
- [交易(Transaction)](#交易transaction)
- [序列化](#序列化)
    - [TxOutPoint](#txoutpoint)
    - [TxIn](#txin)
    - [TxOut](#txout)
    - [交易结构](#交易结构)
- [交易合法性检查](#交易合法性检查)
- [交易费用](#交易费用)
- [coinbase交易](#coinbase交易)
- [交易锁定](#交易锁定)
- [交易目击者](#交易目击者)
- [RBF(Replace-By-Fee)](#rbfreplace-by-fee)
- [序列化](#序列化-1)

<!-- markdown-toc end -->
# 名词解释
## 不定长整型
|取值范围|存储大小|格式|
|:---:|:---:|:---:|
|<0xFD|1|uint8_t|
|<=0xFFFF|3|0xFD+uint16_t|
|<=0xFFFF FFFF|5|0xFE+uint32_t|
|..|9|0xFF+uint64_t|


# 交易(Transaction)
交易代表bitcoin状态机的一次状态迁移. 交易具有以下结构

``` go
type Transaction struct {
    Version int32  //默认为2
    In []TxIn //输入列表
    Out []TxOut //输出列表
    LockTime uint32  //交易锁定
}
```
一个交易有多个输入(TxIn), 也有多个输出(TxOut).

``` go
//代表事务的一个输出
type TxOutPoint struct {
    PreTx [32]byte  //事务ID
    Index uint32 //事务输出的位置索引
}

//代表一个交易输入
type TxIn struct {
    PreOut TxOutPoint
    ScriptSig []byte
    Sequence uint32 //与RBF有关
}

//代表一个交易输出
type TxOut struct {
    Amount int64 //UTXO金额
    ScriptPubKey []byte
}
```
# 序列化
交易数据在传输时需要进行序列化, 序列化后的结构为:

## TxOutPoint

|存储大小|说明|
|:---:|:---:|
|32B|TxOutPoint.PreTx|
|4B|TxOutPoint.Index|

## TxIn
|存储大小|说明|
|:---:|:---:|
|36B|TxIn.PreOut|
|1B+|TxIn.ScriptSig字节数组大小, [不定长整数](#不定长整型)|
|?|TxIn.ScriptSig|
|4B|TxIn.Sequence|
## TxOut
|存储大小|说明|
|:---:|:---:|
|8B|TxOut.Amount|
|1B+|TxOut.ScriptPubKey字节数组大小, [不定长整数](#不定长整型)|
|?|TxOut.ScriptPubKey|

## 交易结构
| 存储大小 | 说明                                                |
|:--------:|:---------------------------------------------------:|
| 4B       | Transaction.Version                                 |
| 0B或者2B | 可选, 若是2B, 则固定为0x0001; 指明是否有witness数据 |
| 1B+      | len(Transaction.In), [不定长整数](#不定长整型)      |
| 41B+     | Transaction.In数组                                  |
| 1B+      | len(Transaction.Out), [不定长整数](#不定长整型)     |
| 9B+      | Transaction.Out数组                                 |
| 0B+      | witness数组                                     |
| 4B       | Transaction.LockTime                                |

# 交易合法性检查
1. 输入必须是有效的: TxIn.PreOut存在, 且TxIn.Index小于TxIn.PreOut的输出长度
2. 输入UTXO的总金额大于或等于输出的UTXO金额
3. 输入UTXO的所有权检查通过
    1. 脚本引擎执行*TxIn.ScriptSig*
    2. 脚本引擎执行*Txs[TxIn.PreOut.PreTx].Out[TxIn.PreOut.Index].ScriptPubKey*
    3. 脚本引擎返回true则通过, false不通过

# 交易费用
输入UTXO的总金额-输出UTXO的总金额

# coinbase交易
 由miner创建的一类特殊的交易:

 - 每个block都有一个coinbase交易
 - coinbase没有输入, 可以凭空创建用于作为其它交易输入的UTXO
 - coinbase交易用于奖励miner, 奖励包含两部分:
     - 交易费用
     - 挖矿激励
 - coinbase的输出UTXO的总金额必须等于交易费用加上挖矿激励
 - 创世块中的coinbase不能用作其它交易的输入

# 交易锁定
一个交易可以通过Transaction.LockTime指定被添加到block的时间:

``` go
var current Block
var tx Transaction
if tx.LockTime == 0 {
    current.Add(tx);
}else if tx.LockTime < 500000000 {
    if current.BlockNumber >= tx.LockTime {
        current.Add(tx);
    }
}else{
    if time.Now().Unix() >= tx.LockTime {
        current.Add(tx);
    }
}
```
# 交易目击者

# RBF(Replace-By-Fee)

# 序列化

# 参考资料
- [Transaction Wiki](https://en.bitcoin.it/wiki/Transaction)
- [Transaction Protocol](https://en.bitcoin.it/wiki/Protocol_documentation#tx)
- [Transaction Expiration](https://en.bitcoin.it/wiki/Transaction_expiration)
- [Segrgated Witness](https://en.bitcoin.it/wiki/Segregated_Witness)
- [Transaction Replacement](https://en.bitcoin.it/wiki/Transaction_replacement)
- [Segregated Witness (Consensus layer)](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki)
