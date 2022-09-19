---
title: Spring Boot集成Mybatis实现多数据源支持
date: 2022-04-21 15:26:34
categories: Java
tags: [Java,Spring Boot,Mybatis]
---
## 1、背景
在实际项目开发过程中，时不时会遇到多数据源的情况，本文详细介绍下Spring Boot集成Mybatis实现多数据源支持。
## 2、集成过程
### 工程结构
首先代码工程结构如下： org.spring.springboot.config.datasource 包含了多数据源的配置，同样有第三个数据源，按照前几个复制即可；resources/mapper目录下面有两个模块，分别是 Mybatis 不同数据源需要扫描的mapper.xml 目录。

```java
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── clang
        │           ├── Application.java
        │               ├── config
        │               │   └── datasource 
        │               │       ├── ClusterDataSourceConfig.java
        │               │       └── MasterDataSourceConfig.java
        │               ├── controller
        │               │   └── UserRestController.java
        │               ├── dao
        │               │   ├── cluster
        │               │   │   └── CityDao.java
        │               │   └── master
        │               │       └── UserDao.java
        │               ├── domain
        │               │   ├── City.java
        │               │   └── User.java
        │               └── service
        │                   ├── UserService.java
        │                   └── impl
        │                       └── UserServiceImpl.java
        └── resources
            ├── application.properties
            └── mapper
                ├── cluster
                │   └── CityMapper.xml
                └── master
                    └── UserMapper.xml
```
### pom.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.clang</groupId>
	<artifactId>springboot-mybatis-mutil-datasource</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>springboot-mybatis-mutil-datasource</name>
	<description>Demo project for Spring Boot</description>

	<!-- springboot -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.8.RELEASE</version>
	</parent>
	<!-- springboot -->

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<mybatis-spring-boot>2.1.4</mybatis-spring-boot>
		<mysql-connector>8.0.11</mysql-connector>
		<druid>1.1.24</druid>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<!-- Spring Boot Mybatis 依赖 -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>${mybatis-spring-boot}</version>
		</dependency>

		<!-- MySQL 连接驱动依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>${mysql-connector}</version>
		</dependency>

		<!-- Druid 数据连接池依赖 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>${druid}</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```
### 配置数据源

```java
## master 数据源配置
master.datasource.url=jdbc:mysql://localhost:3306/db_master?useUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8
master.datasource.username=root
master.datasource.password=root
master.datasource.driverClassName=com.mysql.cj.jdbc.Driver

## cluster 数据源配置
cluster.datasource.url=jdbc:mysql://localhost:3306/db_cluster?useUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8
cluster.datasource.username=root
cluster.datasource.password=root
cluster.datasource.driverClassName=com.mysql.cj.jdbc.Driver
```
### 多数据源配置类

```java
@Configuration
// 扫描 Mapper 接口并容器管理
@MapperScan(basePackages = MasterDataSourceConfig.PACKAGE, sqlSessionFactoryRef = "masterSqlSessionFactory")
public class MasterDataSourceConfig {

    // 精确到 master 目录，以便跟其他数据源隔离
    static final String PACKAGE = "com.clang.dao.master";
    static final String MAPPER_LOCATION = "classpath:mapper/master/*.xml";

    @Value("${master.datasource.url}")
    private String url;

    @Value("${master.datasource.username}")
    private String user;

    @Value("${master.datasource.password}")
    private String password;

    @Value("${master.datasource.driverClassName}")
    private String driverClass;

    @Bean(name = "masterDataSource")
    @Primary
    public DataSource masterDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClass);
        dataSource.setUrl(url);
        dataSource.setUsername(user);
        dataSource.setPassword(password);
        return dataSource;
    }

    @Bean
    @Primary
    public DataSourceTransactionManager masterTransactionManager() {
        return new DataSourceTransactionManager(masterDataSource());
    }

    @Bean(name = "masterSqlSessionFactory")
    @Primary
    public SqlSessionFactory masterSqlSessionFactory(@Qualifier("masterDataSource") DataSource masterDataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(masterDataSource);
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(MasterDataSourceConfig.MAPPER_LOCATION));
        return sessionFactory.getObject();
    }
}
```
@Primary 标志这个 Bean 如果在多个同类 Bean 候选时，该 Bean 优先被考虑。「多数据源配置的时候注意，必须要有一个主数据源，用 @Primary 标志该 Bean」  
@MapperScan 扫描 Mapper 接口并容器管理，包路径精确到 master，为了和下面 cluster 数据源做到精确区分 @Value 获取全局配置文件 application.properties 的 kv 配置,并自动装配 sqlSessionFactoryRef 表示定义了 key ，表示一个唯一 SqlSessionFactory 实例。  
同理可得，从数据源 ClusterDataSourceConfig 配置如下：

```java
@Configuration
@MapperScan(basePackages = ClusterDataSourceConfig.PACKAGE, sqlSessionFactoryRef = "clusterSqlSessionFactory")
public class ClusterDataSourceConfig {

    static final String PACKAGE = "com.clang.dao.cluster";
    static final String MAPPER_LOCATION = "classpath:mapper/cluster/*.xml";

    @Value("${cluster.datasource.url}")
    private String url;

    @Value("${cluster.datasource.username}")
    private String user;

    @Value("${cluster.datasource.password}")
    private String password;

    @Value("${cluster.datasource.driverClassName}")
    private String driverClass;

    @Bean(name = "clusterDataSource")
    public DataSource clusterDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClass);
        dataSource.setUrl(url);
        dataSource.setUsername(user);
        dataSource.setPassword(password);
        return dataSource;
    }

    @Bean(name = "clusterTransactionManager")
    public DataSourceTransactionManager clusterTransactionManager() {
        return new DataSourceTransactionManager(clusterDataSource());
    }

    @Bean(name = "clusterSqlSessionFactory")
    public SqlSessionFactory clusterSqlSessionFactory(@Qualifier("clusterDataSource") DataSource clusterDataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(clusterDataSource);
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(ClusterDataSourceConfig.MAPPER_LOCATION));
        return sessionFactoryBean.getObject();
    }
}
```
上面数据配置分别扫描 Mapper 接口，com.clang.dao.master（对应 xml classpath:mapper/master ） 和 com.clang.dao.cluster（对应 xml classpath:mapper/cluster ） 包中对应的 UserDAO 和 CityDAO 。 都有 @Mapper 标志为 Mybatis 的并通过容器管理的 Bean。Mybatis 内部会使用反射机制运行去解析相应 SQL。
[代码下载](https://gitee.com/chenhu1001/multiple-datasource.git)  
