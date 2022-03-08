# Mybatis 映射结果集 $ 类中类

&emsp;&emsp;在使用SSM(Spring、SpringMvc、Mybatis)架构开发或者含有Mybatis框架的其他开发过程中，常用的结果集映射都是单一对象。
例如：
&emsp;&emsp;1、使用 resultType 指定结果集对象类路径(resultType="com.hy.blogs.UserInfo" 此处指向UserInfo对象);
&emsp;&emsp;2采用 resultMap 指向xml中的resultMap节点(```<resultMap id="userInfoMap" type="com.hy.blogs.UserInfo">...</resultMap>```)。

上述例子中，都是将映射结果匹配到的单一POJO对象UserInfo；如果更复杂一点儿的场景又该如何映射呢，例如下面的问题：

&emsp;&emsp;1、如果某些POJO是内部类又该如何匹配呢？可能有人首先会想到使用 resultType="com.hy.blogs.UserInfo.RoleInfo"(此处RoleInfo为内部类)不可以么？
&emsp;&emsp;2、如果数据库查询结果是嵌套的，比如 RoleInfo 对象只是UserInfo 对象的一个属性，此时又该如何映射呢？
&emsp;&emsp;3、如果数据库查询结果是嵌套的，比如 RoleInfo 对象只是UserInfo 对象的一个属性，并且 RoleInfo 对象是 UserInfo 对象的内部类，此时又该如何映射呢？

PS：针对查询结果字段固定且关系复杂的时候可以采用Java中的 java.util.Map 接口及其衍生类进行接收处理，此时 Map 中的 key 即是 Select 查询结果的字段名称，如果查询时使用了‘AS’，那么 key 就是‘AS’后的别名，否则就是数据库列名。

针对上面的问题，接下来将依次进行解答。

## 一、MyBatis如何将结果映射给内部类？

&emsp;&emsp;如果某些场景下POJO对象B(RoleInfo)是POJO对象A(UserInfo)的内部类，但此时只需查询和对象B(RoleInfo)对应的结果进行返回，这时可以这样做：

### 1、采用 resultType 进行处理

当然，采用这种方法处理肯定是不对的：

```
resultType="com.hy.blogs.entity.UserInfo.RoleInfo"
```

正确的映射对象书写方法为：

```
resultType="com.hy.blogs.entity.UserInfo$RoleInfo"
```

**记住，Mybatis中将查询结果映射到内部类须采用 $ 符号，而不是采用符号‘点’(.)！！**

完整的例子如下：

```
<select id="selectUserRole" resultType="com.hy.blogs.entity.UserInfo$RoleInfo">
     SELECT
    id AS id,
    role_name AS roleName,
    role_desc AS roleDesc,
    ctime AS ctime,
    utime AS utime    
     FROM role_info
</select>
```

### 2、采用 resultMap 进行处理

采用 resultMap 处理肯定需要先定义一个 resultMap 节点，对应的 type 同上面的 resultType 类似，必须采用 $ 进行关联到内部类，如下：

```
<resultMap id="userRoleMap" type="com.hy.blogs.entity.UserInfo$RoleInfo">
    <result column="id" property="id"/>
    <result column="role_name" property="roleName"/>
    <result column="role_desc" property="roleDesc"/>
    <result column="ctime" property="ctime"/>
    <result column="utime" property="utime"/>
</resultMap>
```

对应的查询select中将结果集指向刚刚创建的 resultMap 的 id 即可，写法如下：

```
<select id="selectUserRole" resultMap="userRoleMap">
    SELECT
    id ,
    role_name ,
    role_desc ,
    ctime ,
    utime 
    FROM role_info
</select>
```

## 二、MyBatis如何将查询结果映射给 POJO 对象及其属性外部类？

&emsp;&emsp;平时用得最多的应该是将 MyBatis 查询结果映射给单一的 POJO 对象，当查询结果涵盖某个POJO对象及其属性包含POJO对象的时候该如何映射呢，下面将通过示例进行说明。

&emsp;&emsp;步入正题，咋们将 POJO对象B(RoleInfo) 当作 POJO对象A(UserInfo)的属性，查询结果包含对象A和对象B的属性值。这时，咋们采用 resultMap的方式将结果映射关系处理好，查询的时候直接使用 resultMap=“userRoleMapId”即可。
对应的 resultMap 写法如下：

```
<resultMap id="userRoleMapId" type="com.hy.blogs.entity.UserInfo">
    <result property="id" column="id"/>
    <result property="roleId" column="role_id"/>
    <result property="userId" column="user_id"/>
    <result property="userName" column="user_name"/>
    <result property="userAge" column="user_age"/>
    <result property="userSex" column="user_sex"/>
    <result property="userAddress" column="user_address"/>
    <result property="remark" column="remark"/>
    <result property="ctime" column="ctime"/>
    <result property="utime" column="utime"/>
    <association property="roleInfo" javaType="com.hy.blogs.entity.RoleInfo">
    <result property="id" column="role_id"/>
    <result property="roleName" column="role_name"/>
    <result property="roleDesc" column="role_desc"/>
    <result property="ctime" column="role_ctime"/>
    <result property="utime" column="role_utime"/>
    </association>
</resultMap>
```

如上所示，针对 UserInfo 的角色属性对象 RoleInfo 对象，咋们采用了 association 进行关联。
(PS：association 可以用来处理一对一关联查询、级联查询及嵌套查询等多种场景中！)

&emsp;&emsp;接下来的 select 就很容易了，直接将查询的结果 resultMap 指向 userRoleMapId 即可，如下所示：

```
<select id="selectUserRole" resultMap="userRoleMapId">
    SELECT
    ui.id AS id,
    ui.role_id AS role_id,
    ui.user_id AS user_id,
    ui.user_name AS user_name,
    ui.user_age AS user_age,
    ui.user_sex AS user_sex,
    ui.user_address AS user_address,
    ui.remark AS remark,
    ui.ctime AS ctime,
    ui.utime AS utime,
    ri.id AS role_id,
    ri.role_name AS role_name,
    ri.role_desc AS role_desc,
    ri.ctime AS role_ctime,
    ri.utime AS role_utime
    FROM user_info ui
    LEFT JOIN role_info ri ON(ui.role_id = ri.id)
</select>
```

**注意：resultMap 中的 property 即是对应的 POJO 对象的属性名称，column 则是对应的数据库表列名称，如果查询时使用了 'AS' 则是 ‘AS’ 后面的别名，如果不使用 ‘AS’ 则是查询结果默认的列名。
此处需要注意的是联合查询时不指定别名(采用 AS 或者 空格 均可指定别名)，那么默认的列名是带上对应的数据表别称的列名，如：ui.role_id 对应的列名即是 ui.role_id ，本来想要的列名只是 role_id，但是却带上了数据表的别称 ui；这时只需变换一下写法即可，如：ui.role_id AS role_id 或 ui.role_id role_id 均可实现指定查询列名别称。**

## 三、MyBatis如何将查询结果映射给 POJO 对象及其属性内部类？

&emsp;&emsp;上面第二节说了将查询结果映射给 POJO 对象及其属性类,这一节是将其属性外部类换成属性内部类，大致方法一样，只有少许区别。

&emsp;&emsp;同上面第二节一样，咋们先将查询结果同 POJO 对象的映射关系匹配好，对应的 resultMap 示例如下：

```
<resultMap id="userRoleMapId" type="com.hy.blogs.entity.UserInfo">
    <result property="id" column="id"/>
    <result property="roleId" column="role_id"/>
    <result property="userId" column="user_id"/>
    <result property="userName" column="user_name"/>
    <result property="userAge" column="user_age"/>
    <result property="userSex" column="user_sex"/>
    <result property="userAddress" column="user_address"/>
    <result property="remark" column="remark"/>
    <result property="ctime" column="ctime"/>
    <result property="utime" column="utime"/>
    <association property="roleInfo" javaType="com.hy.blogs.entity.UserInfo$RoleInfo">
        <result property="id" column="role_id"/>
    <result property="roleName" column="role_name"/>
    <result property="roleDesc" column="role_desc"/>
    <result property="ctime" column="role_ctime"/>
    <result property="utime" column="role_utime"/>
    </association>
</resultMap>
```

眼尖的同学也许已经发现不同之处了，那便是 association 节点的 javaType 属性值改动了，外部类写法是 com.hy.blogs.entity.RoleInfo ，内部类的写法则是 com.hy.blogs.entity.UserInfo$RoleInfo ，看着是不是很眼熟，第一节里面所用到的 $ 链接符在这里再次出现了。

&emsp;&emsp;好了,resultMap 写好了映射关系，剩下的就是将 select 的结果指向上面的 resultMap 即可，如下：

```
<select id="selectUserRole" resultMap="userRoleMapId">
    SELECT
    ui.id AS id,
    ui.role_id AS role_id,
    ui.user_id AS user_id,
    ui.user_name AS user_name,
    ui.user_age AS user_age,
    ui.user_sex AS user_sex,
    ui.user_address AS user_address,
    ui.remark AS remark,
    ui.ctime AS ctime,
    ui.utime AS utime,
    ri.id AS role_id,
    ri.role_name AS role_name,
    ri.role_desc AS role_desc,
    ri.ctime AS role_ctime,
    ri.utime AS role_utime
    FROM user_info ui
    LEFT JOIN role_info ri ON(ui.role_id = ri.id)
</select>
```

至此，运用 MyBatis 将查询结果映射给内部类的相关问题已经描述完了，后续有其他问题将继续补充完善！
