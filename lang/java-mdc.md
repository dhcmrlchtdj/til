# MDC logging

---

https://zhuanlan.zhihu.com/p/34175383

---

之前也提过，好奇怎么在 web server 中记录 request Id
今天正好碰到这片文章

---

在传统的线程模型里
可以在收到请求时，生成一个 traceId 放在 ThreadLocal 里面
需要记录日志的时，从 ThreadLocal 里读取即可

数据始终需要传递
不想污染函数签名，不想侵入上层逻辑
能想到的也就全部变量了
这里将整个请求的所有操作都放在一个 thread 里
数据存储在该 thread 的全局变量中也是很自然的

（但这对事件模型就不适用了
（函数式编程里不提倡这种全局变量的做法吧，但感觉在这个场景中其实是适用的

---

文章里的方案很直接，在所有会发生进程切换的地方做一层封装
达成了离开时记录，重入时写回的操作

此外在与上下游服务通信的时候，还是要显示取出 traceId
这在之前的线程方案中也是免不了的

从个人角度讲，感觉这个方案谈不上优雅
其一，traceId 这个事情对上层不够透明（这点还能接受
其二，拦截框架的实现，而不是拦截更底层的机制（未来肯定会变成迁移成本

---

讲 coroutine 时，提到了 CoroutineContext 和 ContinuationInterceptor
有了这些工具，要实现前面提到的“离开时记录，重入时写回”就毕竟简单了

（感慨下 JS 的先天不足。不过想想也有 zone.js 和 async_hooks，还是要找机会实践下