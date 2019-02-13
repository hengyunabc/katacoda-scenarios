
通过`watch`命令可以查看函数的参数/返回值/异常信息。


### 查看UserController的函数返回值

`watch com.example.demo.arthas.user.UserController * returnObj`{{execute T2}}


1. 第一个参数是类名，支持通配
2. 第二个参数是函数名，支持通配


访问 https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/user/1 ,`watch`命令会打印出顺                               

可以输入 `Q`{{execute T2}} 或者 `Ctrl+C` 退出watch命令。

### 返回值表达式

在上面的例子里，第三个参数是`返回值表达式`，它实际上是一个`ognl`表达式，它支持一些内置对象：

* loader
* clazz
* method
* target
* params
* returnObj
* throwExp
* isBefore
* isThrow
* isReturn

你可以利用这些内置对象来组成不同的表达式。比如返回一个数组：

`watch com.example.demo.arthas.user.UserController * '{params[0], target, returnObj}'`{{execute T2}}


更多参考： https://alibaba.github.io/arthas/advice-class.html


### 条件表达式

`watch`命令支持在第4个参数里写条件表达式，比如：

`watch com.example.demo.arthas.user.UserController * returnObj 'params[0] > 100'`{{execute T2}}

当访问 https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/user/1 时，`watch`命令没有输出

当访问 https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/user/101 时，`watch`会打印出结果。

```bash
$ watch com.example.demo.arthas.user.UserController * returnObj 'params[0] > 100'
Press Q or Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:2) cost in 47 ms.
ts=2019-02-13 19:42:12; [cost=0.821443ms] result=@User[
    id=@Integer[101],
    name=@String[name101],
]
```

### 当异常时捕获

`watch`命令支持`-e`选项，表示只捕获抛出异常时的请求：

`watch com.example.demo.arthas.user.UserController * "{params[0],throwExp}" -e`{{execute T2}}


### 按照耗时进行过滤

watch命令支持按请求耗时进行过滤，比如：

`watch com.example.demo.arthas.user.UserController * '{params, returnObj}' '#cost>200'`{{execute T2}}