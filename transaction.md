## 名词解释
### 不定长整型
|取值范围|存储大小|格式|
|:---:|:---:|:---:|
|<0xFD|1|uint8_t|
|<=0xFFFF|3|0xFD+uint16_t|
|<=0xFFFF FFFF|5|0xFE+uint32_t|
|..|9|0xFF+uint64_t|

## 交易
交易代表bitcoin状态机的一次状态迁移.
