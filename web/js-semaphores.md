# js semaphores

---

写爬虫时，经常有这么个问题，怎么控制单个网站的连接数。连接太多就被禁了。
之前用 rxjs 写的时候还是有点蛋疼的。

今天突然想到，这不就是之前看到的 semaphores 吗，可以用 promise 来做很精细的控制呀。