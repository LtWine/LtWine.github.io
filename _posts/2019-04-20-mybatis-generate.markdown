---
layout:     post
title:      "Mybatis Auto Generate"
subtitle:   " \"Good Practice to Auto Generate sql And Model\""
date:       2019-04-20 00:00:00
author:     "Wine"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - mybatis
    - tech
---

> mybatis自动生成model，dao，xml文件

### pom.xml插件配置

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.7</version>
    <dependencies>
        <dependency>
            <groupId> mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.27</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.7</version>
        </dependency>
    </dependencies>
    <configuration>
        <!--允许移动生成的文件 -->
        <verbose>true</verbose>
        <!-- 是否覆盖 -->
        <overwrite>false</overwrite>
        <!-- 自动生成的配置 -->
        <configurationFile>
            src/main/resources/mybatis-generator-mysql.xml</configurationFile>
    </configuration>
</plugin>
```
### mybatis-generator-mysql.xml文件配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="DB2Tables"    targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="fale"/>
            <property name="addRemarkComments" value="true"/>
            <property name="author" value="zhaozb"/>
        </commentGenerator>
        <!--数据库链接地址账号密码,配置开发环境连接-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test_db?serverTimezone=GMT%2B8"
                        userId="uesrname"
                        password="password" >
        </jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!--生成Model类存放位置-->
        <javaModelGenerator targetPackage="com.liumj.model" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!--生成映射文件存放位置-->
        <sqlMapGenerator targetPackage="com.liumj.dbMapper.mysql" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!--生成Dao类存放位置-->
        <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
                type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
                type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
                type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
        -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.liumj.dao" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        <!--生成对应表及类名-->
        <!--
            <table  tableName="test_table" domainObjectName="Test" mapperName="TestDao" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        -->
    </context>
</generatorConfiguration>
```