VarHandle是对一个变量，一类由参数定义的变量(如所有static属性或者所有非static属性)，数组，非堆数据结构的强引用。

对这些变量的访问支持 `plain读写` ，`volatile读写` ，`compare-and-set`

每个AccessMode对应一个accessmodeMethod


访问模式控制原子性和一致性属性。

纯读（get）和写（set）访问保证仅对引用和最多32位的原始类型值是按位原子的，并且不会对执行线程以外的线程施加可观察的排序约束