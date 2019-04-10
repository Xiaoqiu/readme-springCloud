charpter 8

8.1 工作原理

- zuul filter的主要特性：
  - filter 类型
  - filter的执行顺序
  - filter的执行条件
  - filter的执行效果

- zuul提供了一个动态读取，编译，运行这些filter的机制。
- filter直接不直接通向，请求线程通过Request