## 说出ArrayList，Vector，LinkedList的存储性能和特性。

| 对比/种类 | ArrayList | Vector | LinkedList |
| :--- | :--- | :--- | :--- |
| 实现方式 | 数组 | 数组 | 双向链表 |
| 存储容量 | 大于实际元素 | 大于实际元素 | 等于实际元个数 |
| 线程安全 | 不安全 | 安全（利用synchronized） | 不安全 |
| 扩充方式 | 每次增加原来的一半 | 每次增加原来的一倍 |   |



