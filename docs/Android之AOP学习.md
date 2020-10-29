# Android之AOP学习

深入理解Android之AOP [链接](https://blog.csdn.net/innost/article/details/49387395)

大家都知道OOP，即Object Oriented Programming，面向对象编程。而AOP是Aspect Oriented Programming的缩写，中译文为面向切向编程。

* *OOP和AOP都是方法论*。在AOP中，我发现其难度并不在利用AOP干活，而是从AOP的角度来看待问题，设计解决方法。这就是为什么我特意强调*AOP是一种方法论*的原因！
* 在OOP的世界中，问题或者功能都被划分到一个一个的模块里边。每个模块专心干自己的事情，模块之间通过设计好的接口交互。然后并不是所有功能都能够模块化，很多功能都是横跨并嵌入到众多模块中，比如日志埋点，动态权限控制，性能监控(统计函数执行时间)。AOP的目标就是把这些功能集中起来，统一控制和管理。

