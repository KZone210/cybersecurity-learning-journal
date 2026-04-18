# Sqlmap

**自动化盲注，自动猜数据，自动绕过**



## 常用参数

```shell
-u  #指定目标网址
-m  #指定存放多个目标网址的文件路径
-r  #指定存放请求数据包的文件路径
-p  #指定参数注入

-batch-smart  #智能检测并保存（拖库）
--level  #检测等级，等级越高，检测越全面
--level=3  #检测user-agent,referer等http请求头
--risk  #注入等级，等级越高，越容易测出复杂注入，越危险
--risk=2  #加入可能导致数据被更新的Payload
--mobile  #模拟手机访问
--threads  #同时检测线程数

--dbs  #获取所有数据库
--current-user  #获取当前登录用户名
--current-db  #获取当前登录数据库
--is-dba  #当前登录用户是否为管理员

--tables -D 库名  #指定库下有哪些表
--column -T 表名 -D 库名  #指定表下有哪些字段
-T 表名 -C 字段名 --dump  #指定表下的指定字段有哪些数据
--file-read  #读取目标服务器下指定文件
```



## level（测哪里）

### Level 1（默认）

- **测试范围**：仅测 **GET/POST 参数**
- **Payload**：基础、通用、低复杂度
- **速度**：最快
- **场景**：明显、常规注入点（如 `?id=1`）

### Level 2

- **新增**：测试 **Cookie**
- **Payload**：增加数据库特定语句
- **场景**：Cookie 鉴权、会话注入

### Level 3

- **新增**：测试 **User-Agent、Referer** 头部
- **Payload**：更复杂、更多绕过手法
- **场景**：UA 注入、Referer 校验不严

### Level 4

- **新增**：测试 **Host 头部**
- **Payload**：大量边界、编码、混淆 Payload
- **场景**：Host 头注入、强过滤环境

### Level 5（最高）

- **测试范围**：**所有 HTTP 头部（含自定义）**
- **Payload**：最全、最极端、最易触发 WAF
- **速度**：最慢
- **场景**：漏扫、深度渗透、极隐蔽注入点



## risk（测多猛）