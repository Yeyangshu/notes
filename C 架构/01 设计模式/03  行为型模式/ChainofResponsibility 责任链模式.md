# 责任链模式

## 1 责任链模式的定义

责任链模式的英文原话是

> Avoid coupling the sender of a request to its receiver by giving more thanone object a chance to handle the request. Chain the receiving objects andpass the request along the chain until an object handles it.

意思是：使多个对象都有机会处理请求，从而避免了请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它为止。







![image-20201128234752738](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201128234752738.png)

![image-20201129191717957](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201129191717957.png)