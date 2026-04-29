# XSS绕过手段

## 前端绕过

在控制台修改Js代码、从网址栏提交payload、Burp抓包后修改数据包···



## 后端绕过

进行一些测试猜后端代码的过滤逻辑，针对性绕过。列一个payload清单挨个测试

- 双写绕过
- 大小写绕过
- 更换payload标签（script被过滤就换image）



### htmlspecialchars转义

这个函数会把注入payload中特殊符号进行URL编码，使payload失效

对于引号，这个函数默认只编码双引号。所以可以进行测试单引号是否有效



用引号进行测试时

```shell
' onmouseover='alert(1)' 
```

`alert(1)`外层加一层引号和不加引号都试试，浏览器有时候识别不好

**加引号**

![alert加引号](images/10-XSS绕过手段/后端绕过/alert加引号.png)

**不加引号**

![alert不加引号](images/10-XSS绕过手段/后端绕过/alert不加引号.png)











