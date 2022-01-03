# Designing Data-Intensive Applications

## Hash Indexes
1. Hash Indexes 是一个key value mapping. key: e.g. url/id value: address
2. 问题1: key 太多，memory 放不下。 解决办法：存在disk里
3. 问题2: key update 太频繁（写多）。 解决办法：Segement, break logs if it reaches to a limited size, then do & Compaction, throw away duplicate keys in log. 减少存储大小
3. 问题3: 存储格式，csv不好读，一般用binary
3. 问题4: crash。 读每个segement的snapshot