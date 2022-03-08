# JDBC执行MySQL中的存储过程时出现权限错误



> &emsp;&emsp;这个故事要从前几天的中午讲起，午饭吃得正香的时候有同事说线上环境访问失败了，噩耗传来，吃鸡腿都不香了@(・●・)@。想到上午在和运维同学调试运行环境，想必是他变更了啥东西吧。随即打开日志，查看捕获到的错误信息如下：

```java
Caused by: java.sql.SQLException: User does not have access to metadata required to determine stored procedure parameter types. If rights can not be granted, configure connection with "noAccessToProcedureBodies=true" to have driver generate parameters that represent INOUT strings irregardless of actual parameter types.
        at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:998)
        at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:937)
        at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:926)
        at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:872)
        at com.mysql.jdbc.DatabaseMetaData.getCallStmtParameterTypes(DatabaseMetaData.java:1621)
        at com.mysql.jdbc.DatabaseMetaData.getProcedureOrFunctionColumns(DatabaseMetaData.java:3944)
        at com.mysql.jdbc.JDBC4DatabaseMetaData.getProcedureColumns(JDBC4DatabaseMetaData.java:108)
        at com.mysql.jdbc.CallableStatement.determineParameterTypes(CallableStatement.java:808)
        at com.mysql.jdbc.CallableStatement.<init>(CallableStatement.java:598)
        at com.mysql.jdbc.JDBC4CallableStatement.<init>(JDBC4CallableStatement.java:42)
        at sun.reflect.GeneratedConstructorAccessor53.newInstance(Unknown Source)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:422)
        at com.mysql.jdbc.Util.handleNewInstance(Util.java:389)
        at com.mysql.jdbc.CallableStatement.getInstance(CallableStatement.java:500)
        at com.mysql.jdbc.ConnectionImpl.parseCallableStatement(ConnectionImpl.java:3943)
        at com.mysql.jdbc.ConnectionImpl.prepareCall(ConnectionImpl.java:4010)
        at com.mysql.jdbc.ConnectionImpl.prepareCall(ConnectionImpl.java:3986)
        at org.apache.commons.dbcp.DelegatingConnection.prepareCall(DelegatingConnection.java:308)
        at org.apache.commons.dbcp.PoolingDataSource$PoolGuardConnectionWrapper.prepareCall(PoolingDataSource.java:303)
        at org.apache.ibatis.executor.statement.CallableStatementHandler.instantiateStatement(CallableStatementHandler.java:74)
        at org.apache.ibatis.executor.statement.BaseStatementHandler.prepare(BaseStatementHandler.java:82)
        at org.apache.ibatis.executor.statement.RoutingStatementHandler.prepare(RoutingStatementHandler.java:54)
        at org.apache.ibatis.executor.SimpleExecutor.prepareStatement(SimpleExecutor.java:70)
        at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:56)
        at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:259)
        at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:132)
        at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:105)
        at com.github.pagehelper.PageInterceptor.intercept(PageInterceptor.java:142)
        at org.apache.ibatis.plugin.Plugin.invoke(Plugin.java:57)
        at com.sun.proxy.$Proxy173.query(Unknown Source)
        at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:104)
        at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:98)
        at org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(DefaultSqlSession.java:62)
        at sun.reflect.GeneratedMethodAccessor242.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:358)
        ... 63 more
```

> &emsp;&emsp;分析上面的错误结合程序调用数据库的处理得知，用户访问的时候会往数据库写入相关数据，其中会调用一个函数 my_seq(type) 去获取对应业务类型的唯一性ID，现在用户访问失败的原因就是在这一步出错了。

> &emsp;&emsp;经过查阅资料得知，JDBC 在调用存储过程时数据库用户不仅要有 execute 权限，还需要对 mysql.proc 具有查询权限。否则它无法访问 metadata。

> &emsp;&emsp;刚才还运行得好好的，咋出现这种错误了呢，于是便问了运维同学，他说他改了数据库账户权限，给了程序中对应账户的几乎所有权限，包括 execute 权限，那么剩下的就是 mysql.proc 的访问权限问题了。

> &emsp;&emsp;开干，解决掉它吧~

有两种解决方法：

#### 1、根据错误提示，在 jdbc 连接上加“noAccessToProcedureBodies=true”

对应的给 jdbc 连接加上参数，如下：

```SQL
-- noAccessToProcedureBodies 默认是 false
jdbc:mysql://ipaddress:3306/hyblogs?noAccessToProcedureBodies=true  
```

但是呢，加了这个会导致以下问题(由于时间关系，有待一一验证)：

- 调用存储过程时，将没有类型检查，设为字符串类型，并且所有的参数设为in类型，但是在调用registerOutParameter时，不抛出异常；
- 存储过程的查询结果无法使用getXXX(String parameterName)的形式获取，只能通过getXXX(int parameterIndex)的方式获取；

啥？根据错误日志提示的信息进行修改还会导致其他问题，什么鬼？
哈哈，没关系，咋们不是还有第二种方法么！

#### 2、修改对应账户权限，使其可访问 mysql.proc

```SQL
-- 授予对应用户查询 mysql.proc 的权限
-- localhost 可以换成 % 或者 指定IP，如：127.0.0.1，user 换成对应的用户名称，看自己的情况而定
GRANT SELECT ON mysql.proc TO 'user'@'localhost';  

-- 刷新权限
flush privileges;
```

#### 总结

&emsp;&emsp;由于暂不清楚第一种方案是否真的会导致其他问题，而且按照第一种方案还需要修改代码重新发布上线，这样会很麻烦，时间成本比较高，所以这里更推荐第二种方案。

&emsp;&emsp;亲测，我这边写好给数据库用户授权的 SQL 发给运维同学执行完成后用户端就可以正常访问了，这种方式处理起来效率很高。
