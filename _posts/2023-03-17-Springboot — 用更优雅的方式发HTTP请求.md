
最近在开发中遇到了与外部接口请求的业务，发现大佬的代码中用到了一个类`RestTemplate`，特此：找了一篇优秀的文章来学习、封装使用RestTemplate。
让自己再次遇到类似业务能够使用自己的请求工具快速上手熟悉。


## Springboot — 用更优雅的方式发HTTP请求(RestTemplate详解)

RestTemplate是Spring提供的用于访问Rest服务的客户端，RestTemplate提供了多种便捷访问远程Http服务的方法，能够大大提高客户端的编写效率。

我之前的HTTP开发是用apache的HttpClient开发，代发复杂，还得操心资源回收等。代码很复杂，冗余代码多。

本教程将带领大家实现Spring生态内RestTemplate的`Get请求`和`Post请求`还有`exchange指定请求类型`的实践和RestTemplate核心方法源码的分析，看完你就会用优雅的方式来发HTTP请求。

### 1.简述RestTemplate

> 是Spring用于同步client端的核心类，简化了与http服务的通信，并满足RestFul


