Date: 2015-10-29
Title: 发布ADF Essentials 至Glassfish
Tags: Oracle, ADF, glassfish
Slug: adf_to_glassfish


## 软件安装

### JAVA
```
JAVA_HOME=/home/yihan/glassfish/jdk1.8.0_65
export PATH=$JAVA_HOME/bin:$PATH
```
### Glassfish
- Oracle GlassFish Sever is commercially supported version of GlassFish v3. It is recommended for production environments. 
Below is V4  
- 下载: https://glassfish.java.net/download.html
- unzip glassfish-4.1.1*zip
- glassfish4/bin/asadmin start-domain

### ADF Essentials
- 下载： http://www.oracle.com/technetwork/developer-tools/adf/downloads/index.html
	- Oracle ADF Essentials
	- Oracle ADF Faces Components Demo
- 复制到domain目录下的lib目录：/home/yihan/glassfish/glassfish4/glassfish/domains/domain1/lib
- unzip **-j** adf_essentials.zip

## 配置Glassfish

### 更改端口号

- /home/yihan/glassfish/glassfish4/glassfish/domains/domain1/config/domain.xml
```
<network-listener protocol="http-listener-1" port="8080" name="http-listener-1" thread-pool="http-thread-pool" transport="tcp"></network-listener>
          <network-listener protocol="http-listener-2" port="8181" name="http-listener-2" thread-pool="http-thread-pool" transport="tcp"></network-listener>
          <network-listener protocol="pu-protocol" port="4848" name="admin-listener" thread-pool="admin-thread-pool" transport="tcp"></network-listener>
```
、

### 允许远程登陆
`./asadmin --host localhost --port 4848 enable-secure-admin`
### 配置参数
- url: http://localhost:4848 or https://localhost:54848
	- Go to Configurations->Server-config->JVM Settings and choose the JVM Options tab
	- Add the following entries:
	- -XX:MaxPermSize=512m (note this entry should already exist so just make sure it has a big enough value)
	- -Doracle.mds.cache=simple

### 重启
- ./asadmin restart-domain


## 配置数据库

### 添加连接池
```
./asadmin --host localhost --port 54848 create-jdbc-connection-pool --datasourceclassname oracle.jdbc.xa.client.OracleXADataSource --restype javax.sql.XADataSource --property portNumber=1530:password=APPS:user=APPS:serverName=localhost:databaseName=DEMO:connectionAttributes=\; adf_pool
```

### 添加连接
```
./asadmin --host localhost --port 54848 create-jdbc-resource --connectionpoolid adf_pool jdbc/HRDS
```

### 重启



## 添加demo至application

Application->Deploy...->Choose File->Type: Web Application->OK

- - -

## reference documentation

- [https://blogs.oracle.com/shay/entry/deploying_oracle_adf_applications_to](https://blogs.oracle.com/shay/entry/deploying_oracle_adf_applications_to)
- [http://blog-raphaufrj.rhcloud.com/running-adf-essentials-in-tomcat/](http://blog-raphaufrj.rhcloud.com/running-adf-essentials-in-tomcat/)
- [http://docs.oracle.com/cd/E18930_01/html/821-2433/gentextid-110.html#scrolltoc](http://docs.oracle.com/cd/E18930_01/html/821-2433/gentextid-110.html#scrolltoc)
