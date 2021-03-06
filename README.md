# Welcome to MS SQLServer DB support for Liferay CE 7.0
[![Antonio Musarra's Blog](https://img.shields.io/badge/maintainer-Antonio_Musarra's_Blog-purple.svg?colorB=6e60cc)](https://www.dontesta.it)
![travis ci](https://travis-ci.org/amusarra/liferay-portal-sqlserverdb-support.svg?branch=master)
[![Join the chat at https://gitter.im/amusarra/liferay-portal-sqlserverdb-support](https://badges.gitter.im/amusarra/liferay-portal-sqlserverdb-support.svg)](https://gitter.im/amusarra/liferay-portal-sqlserverdb-support?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Twitter Follow](https://img.shields.io/twitter/follow/antonio_musarra.svg?style=social&label=%40antonio_musarra%20on%20Twitter&style=plastic)](https://twitter.com/antonio_musarra)

Those who follow Liferay is aware of the fact that the Community Edition version 7 of Liferay, were eliminated quite a bit of components App Server, Database & Clustering Support. For more detail information you can read the blog post by [Bryan Cheung]( https://www.liferay.com/it/web/bryan.cheung/blog/-/blogs/liferay-portal-7-ce-app-server-database-clustering-support) published on April 7, 2016.

The Liferay 7 CE GA1 no more support OOTB (Out Of The Box):
* Application Server: Oracle WebLogic, IBM WebSphere
* Clustering
* MultiVM Cache
* Oracle Database, Microsoft SQL Server, IBM DB2, Sybase DB

This sample project demonstrates how to add support to the Microsoft SQLServer database. Liferay has performed refactorting the code so that it is possible and easy to add support for databases no longer supported OOTB.

**Attention update**: The driver works with the release GA1, GA2, GA3 and GA4 of the Liferay 7 CE.
[<img src="https://www.dontesta.it/wp-content/uploads/2017/04/PayPalMeAntonioMusarra.png">](https://paypal.me/AntonioMusarra)

## 1. Introduction
To extend support to other databases, Liferay has decided to refactory code to use Java [*SPI (Service Provider Interface)*](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html). SPI is the mechanism that allows you to extend / change the behavior within a system without changing the source. It includes interfaces, classes or methods that the user extends or implements in order to obtain a certain functionality.

In short we must:
* Implement the SPI interface [com.liferay.portal.kernel.dao.db.DBFactory](https://github.com/liferay/liferay-portal/blob/2960360870ae69360861a720136e082a06c5548f/portal-kernel/src/com/liferay/portal/kernel/dao/db/DBFactory.java). Implementation class inside this project is **SQLServerDBFactory.java**
* Implement the abstract class [com.liferay.portal.dao.db.BaseDB](https://github.com/liferay/liferay-portal/blob/master/portal-impl/src/com/liferay/portal/dao/db/BaseDB.java) for SQL Server DB. Implementation class inside this project is **SQLServerDB.java**

The following code shows how service providers are loaded via SPI.
```
public DBManagerImpl() {
  ServiceLoader<DBFactory> serviceLoader = ServiceLoader.load(
    DBFactory.class, DBManagerImpl.class.getClassLoader());

  for (DBFactory dbFactory : serviceLoader) {
    _dbFactories.put(dbFactory.getDBType(), dbFactory);
  }
}
```
To register your service provider, you create a provider configuration file, which is stored in the **META-INF/services** directory of the service provider's JAR file. The name of the configuration file is the fully qualified class name of the service provider, in which each component of the name is separated by a period (.), and nested classes are separated by a dollar sign ($).

The provider configuration file contains the fully qualified class names (FQDN) of your service providers, one name per line. The file must be UTF-8 encoded. Additionally, you can include comments in the file by beginning the comment line with the number sign (#).

Our file is called com.liferay.portal.kernel.dao.db.DBFactory and contain the FQDN of the class [it.dontesta.labs.liferay.portal.dao.db.SQLServerDBFactory](https://github.com/amusarra/liferay-portal-sqlserverdb-support/blob/master/src/main/java/it/dontesta/labs/liferay/portal/dao/db/SQLServerDBFactory.java)


In the figure below shows the complete class diagram for SQLServerDB.

![Class Diagram for SQLServerDB](https://www.dontesta.it/wp-content/uploads/2014/02/SQLServerDB.png)

## 2. Build project
Requirements for build project
1. Sun/Oracle JDK 1.8
2. Maven 3.x (for build project) or Gradle 2.x

The driver that adds support for SQL Server database is a jar (**liferay-portal-sqlserverdb-support-${version}.jar**) which then will be installed in ROOT/WEB-INF/lib (for apache tomcat).

To generate the driver for SQL Server database just follow the instructions below.

You can download the binary jar [liferay-portal-sqlserverdb-support-1.0.1-SNAPSHOT.jar](https://github.com/amusarra/liferay-portal-sqlserverdb-support/releases/download/v1.0.1/liferay-portal-sqlserverdb-support-1.0.1-SNAPSHOT.jar), by doing so you can avoid doing the build.

```
$ git clone https://github.com/amusarra/liferay-portal-sqlserverdb-support.git
$ cd liferay-portal-sqlserverdb-support
$ mvn package
```

the build process create the jar inside the (maven) target directory:

```
liferay-portal-sqlserverdb-support-1.0.1-SNAPSHOT.jar
```

If you have a Gradle build system, then you can build jar by the following command

```
$ git clone https://github.com/amusarra/liferay-portal-sqlserverdb-support.git
$ cd liferay-portal-sqlserverdb-support
$ gradle build
```

the build process create the jar inside the build/libs directory.

## 3. Install Liferay CE 7 on Microsoft SQLServer Database

To install Liferay on SQLServer you must have previously configured a schema for Liferay on an SQLServer instance (including SQLServer Express Edition).

I have used an SQLServer Express 2014 installation on Windows 8.1 created on the fly on VirtualBox.

The parameters of my SQLServer instance are:
* Username: liferay
* Password: liferay12345
* DB Name: liferayce7
* FQDN: sqlserverdb.vm.local (IP: 192.168.56.101)
* TCP/IP Port: 1433
* Instance ID: SQLEXPRESS

For the installation of Liferay follow the following steps:

1. Download [Liferay CE 7 GA4 Tomcat Bundle](https://sourceforge.net/projects/lportal/files/Liferay%20Portal/7.0.3%20GA4/liferay-ce-portal-tomcat-7.0-ga4-20170613175008905.zip/download) from sourceforge
2. Extract the Liferay bundle (in my case $LIFERAY_HOME is /opt/liferay-ce-portal-7.0-ga4)
3. Copy the jar **liferay-portal-sqlserverdb-support-${version}.jar** in $LIFERAY_HOME/$TOMCAT_HOME/webapps/ROOT/WEB-INF/lib
4. Download and install [Microsoft JDBC driver per SQLServer](https://msdn.microsoft.com/it-it/library/mt683464.aspx) in $LIFERAY_HOME/$TOMCAT_HOME/lib/ext
5. Create the **portal-ext.properties** in $LIFERAY_HOME with the content as the file below. ***You should modify the JDBC connection parameters to the your db and the value of liferay.home***
6. Launch the Liferay Portal through the command $LIFERAY_HOME/$TOMCAT_HOME/bin/startup.sh
7. See the Liferay activities via the log file $LIFERAY_HOME/$TOMCAT_HOME/logs/catalina.out

Below you can see the portal-ext.properties

```
##
## Admin Portlet
##
    #
    # Configure email notification settings.
    #
    admin.email.from.name=Joe Bloggs
    admin.email.from.address=test@liferay.com

##
## JDBC
##

    #
    # SQL Server
    #
    jdbc.default.driverClassName=com.microsoft.sqlserver.jdbc.SQLServerDriver
    jdbc.default.username=liferay
    jdbc.default.password=liferay12345
    jdbc.default.url=jdbc:sqlserver://sqlserverdb.vm.local;databaseName=liferayce7

##
## Liferay Home
##
    #
    # Specify the Liferay home directory.
    #
    liferay.home=/opt/liferay-ce-portal-7.0-ga4

##
## Setup Wizard
##
    #
    # Set this property to true if the Setup Wizard should be displayed the
    # first time the portal is started.
    #
    setup.wizard.enabled=false
```

![Liferay CE 7 GA1 on SQLServer DB ](https://www.dontesta.it/wp-content/uploads/2014/02/Liferay7CEOnSQLServerDB.png)

![Liferay CE 7 GA1 on SQLServer DB ](https://www.dontesta.it/wp-content/uploads/2014/02/Liferay7CEOnSQLServerDB_1.png)
