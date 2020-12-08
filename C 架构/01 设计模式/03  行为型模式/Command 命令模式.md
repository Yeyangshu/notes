# 命令模式

封装命令

结合cor实现undo



命令模式（Command Pattern）又称为行动（Action）模式或交易（Transaction）模式。

1 命令模式的定义

命令模式的英文原话是：

> Encapsulate a request as an object, thereby letting you parameterize clientswith different requests, queue or log requests, and support undoableoperations.

意思是：将一个请求封装成一个对象，从而让你使用不同的请求把客户端参数化，对请求排队或者记录请求日志，可以提供命令的撤销和恢复功能。

## 解析

别名：Action/ Transaction

宏命令：命令+组合

多次undo：命令+责任链

transaction回滚