# upload-labs通关思路

## Pass-01

检查是否JS前端过滤，或者直接关闭Javascript

![01](images/13-upload-labs通关思路/Pass-01/01.png)

![02](images/13-upload-labs通关思路/Pass-01/02.png)

可能是这个函数搞的鬼，干掉它！

![03](images/13-upload-labs通关思路/Pass-01/03.png)

上传成功

这里换了火狐浏览器，因为谷歌浏览器干掉函数之后没有效果



## Pass-02

检查是否JS前端过滤，或者直接关闭Javascript

![01](images/13-upload-labs通关思路/Pass-02/01.png)

这里有一个可以参数，先干掉它试试

![02](images/13-upload-labs通关思路/Pass-02/02.png)

还是不行，那就直接关闭Javascript试试

![03](images/13-upload-labs通关思路/Pass-02/03.png)

![04](images/13-upload-labs通关思路/Pass-02/04.png)

Javascript已经停用了，但是还上传失败，说明大概率是后端搞的鬼

![05](images/13-upload-labs通关思路/Pass-02/05.png)

抓包改一下`Content-Type`试试

![06](images/13-upload-labs通关思路/Pass-02/06.png)

上传成功了

![07](images/13-upload-labs通关思路/Pass-02/07.png)

服务端`upload`文件夹

![08](images/13-upload-labs通关思路/Pass-02/08.png)



## Pass-03

这关上来直接禁用JS，但是上传失败，是后端在搞鬼

![01](images/13-upload-labs通关思路/Pass-03/01.png)

它既然明确写出不允许上传的文件类型，可能是后端做了白名单或黑名单限制。上传一个小众类型的文件，测试后端是黑名单过滤还是白名单过滤。如果上传成功，说明后端是针对某些后缀名的文件做了黑名单过滤，那么就可以尝试上传它没有指定过滤的文件绕过检测机制

![02](images/13-upload-labs通关思路/Pass-03/02.png)

上传成功了，表示后端使用的是黑名单过滤机制

![03](images/13-upload-labs通关思路/Pass-03/03.png)

![04](images/13-upload-labs通关思路/Pass-03/04.png)

尝试上传一个`.phtml`结尾的phpinfo文件，如果服务端开启了对应解析功能，就可以解析文件中的php代码

![05](images/13-upload-labs通关思路/Pass-03/05.png)

文件上传成功了

![06](images/13-upload-labs通关思路/Pass-03/06.png)

![07](images/13-upload-labs通关思路/Pass-03/07.png)

如果我们是渗透人员，无法访问服务端拿保存路径。可以直接复制图片地址进行访问。

```text
http://192.168.125.162/upload/upload/202605030249196835.phtml
```

![08](images/13-upload-labs通关思路/Pass-03/08.png)

这时已经访问到了我们上传的phpinfo文件，但是文件代码并没有被执行。那是因为服务端Apache没有开启对应功能

![09](images/13-upload-labs通关思路/Pass-03/09.png)

在Apache的配置文件中开启第对应功能

```text
AddType application/x-httpd-php .php .phtml
```

![10](images/13-upload-labs通关思路/Pass-03/10.png)

![11](images/13-upload-labs通关思路/Pass-03/11.png)

**保存并重启**服务后，重新访问刚才上传的phpinfo文件

![12](images/13-upload-labs通关思路/Pass-03/12.png)

这时我们上传的phpinfo文件为解析执行

在配置文件中添加可以被解析的文件类型，如php3、php5···保存并重启服务

![13](images/13-upload-labs通关思路/Pass-03/13.png)

上传后缀名为`.php5`结尾的phpinfo文件，发现文件可以正常访问并被解析

![14](images/13-upload-labs通关思路/Pass-03/14.png)

![15](images/13-upload-labs通关思路/Pass-03/15.png)

**如果运维人员疏忽大意，开启或者测试后忘记关闭AddType功能，那么服务端就可以解析除了.php文件结尾的其他php代码文件，非常危险**



## Pass-04

关闭JS后不能上传`.php`结尾的文件，但是可以上传`.gif`结尾的文件，大概率时黑名单过滤

![01](images/13-upload-labs通关思路/Pass-04/方法1/01.png)

![02](images/13-upload-labs通关思路/Pass-04/方法1/02.png)

![03](images/13-upload-labs通关思路/Pass-04/方法1/03.png)

![04](images/13-upload-labs通关思路/Pass-04/方法1/04.png)



### 方法1

尝试上传图片马

![05](images/13-upload-labs通关思路/Pass-04/方法1/05.png)

![06](images/13-upload-labs通关思路/Pass-04/方法1/06.png)

访问图片马

![07](images/13-upload-labs通关思路/Pass-04/方法1/07.png)

图片马并没有被当成php文件解析

![08](images/13-upload-labs通关思路/Pass-04/方法1/08.png)

上传`.htaccess`文件，这个文件可以覆盖掉当前目录下的Apache配置文件，让当前目录下的所有文件都被当成php文件解析

```text
<FilesMatch "">
SetHandler application/x-httpd-php
</FilesMatch>
```

![09](images/13-upload-labs通关思路/Pass-04/方法1/09.png)

![10](images/13-upload-labs通关思路/Pass-04/方法1/10.png)

这时图片马被当作php文件解析

![11](images/13-upload-labs通关思路/Pass-04/方法1/11.png)

服务端Apache配置文件需要开启对应功能，上传的`.htaccess`文件才有效

![12](images/13-upload-labs通关思路/Pass-04/方法1/12.png)

还有一种覆盖配置文件的方法，上传`.user.ini`文件。可以覆盖当前目录下的php配置文件，使指定文件被包含在当前目录下的php文件中解析。Apache和Nginx通用。

```shell
.user.ini的两种写法，一般包含在文件头以防止服务端php文件中出现 exit
auto_prepend_file = webshell.jpg(指定需要被包含的文件)  #包含在文件头
auto_append_file = <filename>  #包含在文件尾
```

- 服务端不对上传的文件改名
- 存放上传文件的目录下必须有一个php文件

![13](images/13-upload-labs通关思路/Pass-04/方法1/13.png)

![14](images/13-upload-labs通关思路/Pass-04/方法1/14.png)



#### 原理

先上传一个`.jpg`结尾的图片马，再上传一个`.htaccess`文件覆盖当前目录下的Apache配置文件，使当前目录下的所有文件都被当做php文件解析。不足之处在于：

- 服务端Apache配置文件需要开启`Allowoverride`功能，上传的`.htaccess`文件才有效
- 服务端不对上传的文件改名，否则`.htaccess`文件失效



### 方法2

用Burpsuite抓一个上传`webshell.php`的数据包，把文件后缀名改成`.php:.jpg`，`Content-type`改成`image/png`

![01](images/13-upload-labs通关思路/Pass-04/方法2/01.png)

放行数据包，文件上传成功，但是文件中没有数据

![02](images/13-upload-labs通关思路/Pass-04/方法2/02.png)

![03](images/13-upload-labs通关思路/Pass-04/方法2/03.png)

再抓一个上传`webshell.php`的数据包，把文件后缀名改成`.<<<`表示把数据输入到这个文件中

![04](images/13-upload-labs通关思路/Pass-04/方法2/04.png)

![05](images/13-upload-labs通关思路/Pass-04/方法2/05.png)

访问`webshell.php`

![06](images/13-upload-labs通关思路/Pass-04/方法2/06.png)



#### 原理

直接在Windows中创建文件，不能加`:`，但是在php+Windows场景下，可以利用先上传一个后缀名为`.php:.jpg`的文件。文件通过php对文件后缀名的检测被传到Windows系统中时，由于Windows中的文件后缀名不能出现`:`，所以Windows系统会把`:.jpg`去掉。这种方法的不足之处在于文件大小会变成0kb，也就是文件数据会被清空



### IIS解析图片马

IIS服务器中的`asp.asp;.jpg`文件能够直接被解析成asp代码，因为IIS服务器读取文件名是从左到右进行读取，当读到`.asp`时直接当成asp文件解析

![01](images/13-upload-labs通关思路/Pass-04/IIS解析图片马/01.png)

![02](images/13-upload-labs通关思路/Pass-04/IIS解析图片马/02.png)



## Pass-05(Windows)

这关的黑名单限制很多，但是黑名单方法很难规避漏洞

![01](images/13-upload-labs通关思路/Pass-05/01-大小写绕过/01.png)

Windows对大小写限制不严格，可以尝试大小写绕过

![02](images/13-upload-labs通关思路/Pass-05/01-大小写绕过/02.png)

上传成功

![03](images/13-upload-labs通关思路/Pass-05/01-大小写绕过/03.png)

访问`webshell.pHP`

![04](images/13-upload-labs通关思路/Pass-05/01-大小写绕过/04.png)



## Pass-06(Windows)

这关针对Windows系统特性，Windows系统文件名最后不允许出现`.`。`.php. `文件通过php检测被上传到系统中时Windows系统会把文件名最后的`.`去掉，实际被保存的文件名为`webshell.php`

![01](images/13-upload-labs通关思路/Pass-06/01.png)

先上传`webshell.php`，抓包后在文件名后面加空格和`.`

![03](images/13-upload-labs通关思路/Pass-06/03.png)

![04](images/13-upload-labs通关思路/Pass-06/04.png)

访问`webshell.php`

![05](images/13-upload-labs通关思路/Pass-06/05.png)















