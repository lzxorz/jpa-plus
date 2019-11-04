### jpa-plus

关于jpa功能增强的开源项目，在github上可以找到几个，比如：[spring-data-jpa-extra](https://github.com/slyak/spring-data-jpa-extra)、[jpa-spec](https://github.com/wenhao/jpa-spec)、[spring-data-jpa-expansion](https://github.com/fast-family/spring-data-jpa-expansion)。

我原本拿`spring-data-jpa-extra`魔改了一番，但是后来觉得封装过于复杂，不够简单高效，弃用了。 

然后想按照`myBatis-plus`的`QueryWrapper`和`UpdateWrapper`自己封装出来jpa版本，封装`QueryWrapper`很容易想到可以使用jpa的`Specification`实现。但是，想实现`UpdateWrapper`的效果就有点儿...。封装`QueryWrapper`块的时候遇到点儿问题，在查百度时看到了`jpa-spec`这个项目。此时，我也差不多封装完了、绝不放弃。顺利写完我的版本(更接近`myBatis-plus`的`QueryWrapper`的写法)。后来觉得联查功能不够强大，并且必须在实体类中加响应的注解(orm...我不喜欢)，总之还是不够简单高效，又弃用了。

经过查资料多次尝试，发现NativeSql查询很好很强大，实现ResultTransformer可以定制查询结果转化到任意pojo，应该可以封装出我想要的`简单高效`的效果，于是又一次的努力，有了`jpa-plus`。目前，只封装了查询功能。更新功能还没封装。

### jpa-plus使用说明

sevice层的方法中
```java
NativeSqlQuery nativeSql = NativeSqlQuery.builder()
    .select(查询的列) //必有
    .from(表名 AS 表别名 LEFT JOIN 表名 AS 表别名 ON XXX = YYY) //必有
    // where 条件， eq/ne/lt/...可以出现0～n次，生成SQL时 默认用 AND 连接多个where条件
    // 自动判断属性不为null，才会生成对应sql片段
    .eq(表别名.列名, java属性值/常量值)
    .ne(表别名.列名, java属性值/常量值)
    .lt(表别名.列名, java属性值/常量值)
    .lte(表别名.列名, java属性值/常量值)
    .gt(表别名.列名, java属性值/常量值)
    .gte(表别名.列名, java属性值/常量值)
    .isNull(表别名.列名) // 不需要传值
    .isNotNull(表别名.列名) // 不需要传值
    .startsWith(表别名.列名, java属性值/常量值)
    .contains(表别名.列名, java属性值/常量值)
    .endsWith(表别名.列名, java属性值/常量值)
    .notStartsWith(表别名.列名, java属性值/常量值)
    .notContains(表别名.列名, java属性值/常量值)
    .notEndsWith(表别名.列名, java属性值/常量值)
    .in(表别名.列名, java属性值/常量值)         // 传入Array或List
    .notIn(表别名.列名, java属性值/常量值)      // 传入Array或List
    .between(表别名.列名, java属性值/常量值)    // 传入Array或List, 长度应该是2
    .notBetween(表别名.列名, java属性值/常量值) // 传入Array或List, 长度应该是2
    .sqlStrPart(自定义sql字符串片段) //追加到where条件尾部(数据权限sql片段)
    // 以下这些 也是可有可无，跟原生sql写法别无二致
    .groupBy(表别名.列名)
    .having(表别名.列名 操作符 值 [AND/OR] [表别名.列名 操作符 值])
    .orderBy(表别名.列名 ASC[,表别名.列名 DESC])
    // build可以不写, 照顾用习惯lombok的人(不build一下,可能浑身不自在)
    .build();

    // nativeSql
    // pojo.class  普通的java类即可，列名(下划线连接==自动转换==>驼峰命名，去匹配java类的属性)
    // pageRequest 需要分页,可传入. pageRequest对象中有排序的Sort数据会无视(请用orderBy()排序)
    // 返回List<pojo>或Page<pojo>
    dao.findAllByNativeSql(nativeSql, JobLog.class, pageRequest);
```

### jpa-plus用法示例

sevice层的方法中
```java
 
    Object createTime = jobLog.getParams("create_time"); //传进来是创建起止时间的数组

    NativeSqlQuery nativeSql = NativeSqlQuery.builder()
        .select("tjl.*")
        .from("t_job_log tjl")
        .eq("tjl.bean_name", jobLog.getBeanName())
        .eq("tjl.method_name", jobLog.getMethodName())
        .contains("tjl.parameter", jobLog.getParameter())
        .eq("tjl.status", jobLog.getStatus())
        .between("date_format(tjl.create_time,'%Y-%m-%d')", createTime)
        .orderBy("tjl.id")
        .build();

    return dao.findAllByNativeSql(nativeSql, JobLog.class, pageRequest);
    
```