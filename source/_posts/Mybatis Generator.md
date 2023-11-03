根据数据库的表自动生成相应的实体类、mapper接口及对应的xml文件。

1.pom.xml中添加插件和依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.3</version>
	</dependency>
</dependencies>
 
<build>
    <plugins>
    	<plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.7</version>
            <configuration>
                <!--mybatis的代码生成器的配置文件-->
                <configurationFile>
                    src/main/resources/generatorConfig.xml
                </configurationFile>
                <!--允许覆盖生成的文件-->
                <!--有时候我们的数据库表添加了新字段，需要重新生成对应的文件。
                    常规做法是手动删除旧文件，然后在用 MyBatis Generator 生成新文件。
                    当然你也可以选择让 MyBatis Generator 覆盖旧文件，省下手动删除的步骤。-->
                <!--值得注意的是，MyBatis Generator 只会覆盖旧的 po、dao、而 *mapper.xml 不会覆盖，
                    而是追加，这样做的目的是防止用户自己写的 sql 语句一不小心都被 MyBatis Generator 给覆盖了-->
                <overwrite>true</overwrite>
                <verbose>true</verbose>
                <!--将当前pom的依赖项添加到生成器的类路径中-->
                <!--<includeCompileDependencies>true</includeCompileDependencies>-->
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>8.0.19</version>
                </dependency>
                <dependency>
                    <groupId>org.mybatis.generator</groupId>
                    <artifactId>mybatis-generator-core</artifactId>
                    <version>1.4.0</version>
                </dependency>
            </dependencies>
    	</plugin>
    </plugins>
</build>
```

2.resources目录下配置Mybatis的config文件(Spring Boot项目只需在application.yml文件中配置即可)

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"></properties>

    <typeAliases>
        <package name=""/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!--设置连接数据库的驱动-->
                <property name="driver" value="${jdbc.driver}"/>
                <!--设置连接数据库的连接地址-->
                <property name="url" value="${jdbc.url}"/>
                <!--设置连接数据库的用户名-->
                <property name="username" value="${jdbc.username}"/>
                <!--设置连接数据库的密码-->
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <!--引入MyBatis映射文件-->
    <mappers>
        <!--package方式需要将mapper处于同一文件夹下-->
        <package name="com.xwy.core.mapper"/>
<!--        <mapper resource="mapper/UserInfoMapper.xml"></mapper>-->
    </mappers>
</configuration>
```

jdbc.properties

```
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test1?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
jdbc.username=root
jdbc.password=12abCD##
```

3.resources目录下配置generator的xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 引入配置文件 -->
    <!--<properties resource="jdbc.properties"></properties>-->

    <!-- 目标数据库 -->
    <!-- 一个数据库一个context, context子元素必须按照如下顺序
        property*、plugin*、commentGenerator?、jdbcConnection、javaTypeResolver?
        javaModelGenerator、sqlMapGenerator?、javaClientGenerator?、table+
    -->
    <!--id : 随便填，保证多个 context id 不重复就行
        defaultModelType ： 可以不填，默认值 conditional，flat表示一张表对应一个po
        targetRuntime ：可以不填，默认值 MyBatis3，常用的还有 MyBatis3Simple，这个配置会影响生成的 dao 和 mapper.xml的内容
        targetRuntime = MyBatis3Simple，生成的 dao 和 mapper.xml，接口方法会少很多，只包含最最常用的
    -->
    <context id="Mysql" targetRuntime="Mybatis3">

        <property name="javaFileEncoding" value="UTF-8"/>

        <!-- 生成的entity，将implements Serializable -->
<!--        <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />-->
        <!-- 为生成的entity创建一个toString方法 -->
<!--        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>-->
        <!-- 生成的entity，增加了equals 和 hashCode方法-->
<!--        <plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin" />-->
        <!--生成mapper.xml时覆盖原文件-->
<!--        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />-->

        <!-- 自定义注释 -->
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="false"/>
            <!--添加 db 表中字段的注释-->
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>

        <!-- 数据库连接信息 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test1?serverTimezone=UTC"
                        userId="root"
                        password="12abCD##">
            <!--高版本的 mysql-connector-java 需要设置 nullCatalogMeansCurrent=true-->
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>

        <javaTypeResolver>
            <!--类型解析器-->
            <!-- 默认false，把jdbc decimal 和 numeric 类型解析为integer -->
            <!-- true，把jdbc decimal 和 numeric 类型解析为java.math.bigdecimal-->
            <property name="forceBigDecimals" value="false"/>
            <!--默认false
                false，将所有 JDBC 的时间类型解析为 java.util.Date
                true，将 JDBC 的时间类型按如下规则解析
                   DATE	                -> java.time.LocalDate
                   TIME	                -> java.time.LocalTime
                   TIMESTAMP                   -> java.time.LocalDateTime
                   TIME_WITH_TIMEZONE  	-> java.time.OffsetTime
                   TIMESTAMP_WITH_TIMEZONE	-> java.time.OffsetDateTime
            -->
            <property name="useJSR310Types" value="false"/>
        </javaTypeResolver>

        <!-- java实体类路径 -->
        <javaModelGenerator targetPackage="com.xwy.core.entity" targetProject=".\src\main\java">
            <!-- 是否让schema作为包后缀-->
            <property name="enableSubPackages" value="false" />
            <!-- 从数据库返回的值被清理前后的空格-->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- 生成映射文件xml的包名和位置 -->
        <sqlMapGenerator targetPackage="mapper" targetProject=".\src\main\resources">
            <!-- 是否让schema作为包后缀-->
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- 生成Mapper接口的包名和位置
            type="XMLMAPPER" 会将接口的实现放在 mapper.xml中，也推荐这样配置。
            type="ANNOTATEDMAPPER"，接口的实现通过注解写在接口上面
        -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.xwy.core.mapper" targetProject=".\src\main\java">
            <!-- 是否让schema作为包后缀-->
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>

        <!-- 用于自动生成代码的数据库表；生成哪些表;
            schema为数据库名，oracle需要配置，mysql不需要配置。
            tableName为对应的数据库表名
            domainObjectName 是要生成的实体类名(可以不指定)（其中 domainObjectName 不配置时，它会按照帕斯卡命名法将表名转换成类名）
            enableXXXByExample 默认为 true， 为 true 会生成一个对应Example帮助类，帮助你进行条件查询，不想要可以设为false
            生成全部表tableName设为 %
        -->
        <table tableName="%" enableCountByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               enableUpdateByExample="false"/>

        <!--<table schema="gwshopping" tableName="pms_product" domainObjectName="PmsProduct" enableCountByExample="true"
               enableDeleteByExample="true" enableSelectByExample="true"
               enableUpdateByExample="true">
        </table>-->
    </context>
</generatorConfiguration>
```



ps:不好用，不如直接用mybatis-plus生成