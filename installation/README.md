# Installation

<!-- toc -->

## Software requirements

Orbeon Forms runs on any platform that supports:

* Java 6 (not recommended), 7, or 8
* a Servlet 2.5 (or greater) container such as [Apache Tomcat](http://tomcat.apache.org/). For versions supported, see [Tomcat](tomcat.md).

## Hardware requirements

We recommend you run Orbeon Forms on a dedicated server or instance that satisfies the following requirements:

* CPU: recent 4-core, or more, Intel Xeon or Core i7. We don't recommend AMD CPUs, which, as of 2015, tend to be 2-4 times slower than Intel CPUs per core.
* RAM: 4 GB of RAM, or more, available to the Java Virtual Machine (JVM heap size).

If using AWS EC2, we recommend you start with a c4.2xlarge instance. For most projects, it is safe to start with a configuration along those lines, but you might want to have more powerful or multiple servers or instances (or equivalent) for situations calling for high availability, or to handle more load. When running Orbeon Forms PE on multiple servers or instances, you need one [PE subscription](http://www.orbeon.com/pricing) per server or instance. 

For more details on sizing, see the section on [how much load Orbeon Forms can handle](../faq/form-builder-runner.md#how-much-load-can-orbeon-forms-handle).

## Java virtual machine configuration

Configure the Java VM with:

* `-Xmx` option for dedicated Java heap memory:
    * on a development machine: at least 1 GB of Java heap: `-Xmx1g`
    * on a production machine: at least 4 GB of Java heap: `-Xmx4g`
* ` -XX:MaxPermSize` for "permgen" space (Java 1.7):
    * use at least: `-XX:MaxPermSize=256m`

Also, make sure that you do *not* have tiered compilation when using Java 7. See [A dangerous Java 7 JVM option: TieredCompilation
](http://blog.orbeon.com/2015/08/a-dangerous-java-7-jvm-option.html).

*NOTE: On Unix systems, GIJ / GCG is not supported as there are reports of issues with that runtime environment and Orbeon Forms. Instead, we recommend you use the Oracle runtime Java environment.*

## Database setup

Out-of-the-box, forms you create with Form Builder, as well as data captured with those forms, will be saved in an embedded database called eXist. You can setup Orbeon Forms so this data gets [stored in your relational database](../form-runner/persistence/relational-db.md), but if you're getting started with Orbeon Forms, you might to just use the embedded eXist, even if just temporarily.

Note that eXist will need to be able to write to the `WEB-INF/exist-data` directory, wherever Orbeon Forms `.war` file is uncompressed. So, especially if you're on UNIX, make sure that this directory is writable by the process running your app server.

## License installation (Orbeon Forms PE only)

* If you are running Orbeon Forms CE, you don't need to install a license file.
* If you are running Orbeon Forms PE:
    * complete the steps for your application server below
    * you can obtain a full licence from Orbeon, or get a [trial license](http://demo.orbeon.com/orbeon/fr/orbeon/register/new)
    * before starting your servlet container, copy your license file under the Orbeon Forms WAR file as:
    ```
    WEB-INF/resources/config/license.xml
    ```

With Orbeon Forms 4.1 and newer, you can also place license.xml file under the user's home directory. For example, on Unix systems:

```
~/.orbeon/license.xml
```

Orbeon Forms first searches for the license file within the WAR, and if not found attempts to find it under the home directory.

The benefit of this approach is that you don't have to find where the WAR file is deployed in your container, or to uncompress and recompress the WAR file with the license.

_NOTE:  Orbeon Forms uses Java's `System.getProperty("user.home")` to identify the user's home directory.__  This corresponds to the user running the servlet container and not necessarily to the user of the developer or system administrator._

## Base URL for internal services

This step is sometimes optional.

Depending on your setup, if things don't work out of the box (for example if you have database errors with the sample forms) you might have to set the [oxf.url-rewriting.service.base-uri](../configuration/properties/general.md#oxfurl-rewritingservicebase-uri) configuration property in your `properties-local.xml` file.

Often, it is enough to set it to the following (adjusting for port and prefix):

```xml
<property
    as="xs:anyURI"
    name="oxf.url-rewriting.service.base-uri"
    value="http://localhost:8080/orbeon"/>
```

For more information about how to set configuration properties, see [Configuration Properties](../configuration/properties/README.md).

## Logging configuration

This step is optional.

Orbeon Forms has a logging configuration file under `WEB-INF/resources/config/log4j.xml`. By default, logging information is output to a file path relative to the directory where you start your application server.

```xml
<appender name="SingleFileAppender" class="org.apache.log4j.FileAppender">
    <param name="File" value="../logs/orbeon.log"/>
    <param name="Append" value="false" />
    <param name="Encoding" value="UTF-8"/>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{ISO8601} %-5p %c{1} %x - %m%n"/>
    </layout>
</appender>
```

You can change this by modifying the file parameter. Notes that on Windows, you must use forward slashes:

```xml
<appender name="SingleFileAppender" class="org.apache.log4j.FileAppender">
    <param name="File" value="C:/My Path/To/Logs/orbeon.log"/>
    <param name="Append" value="false" />
    <param name="Encoding" value="UTF-8"/>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{ISO8601} %-5p %c{1} %x - %m%n"/>
    </layout>
</appender>
```

The benefit of changing this configuration is that you know exactly where the file is stored. This can be really handy when trying to troubleshoot issues.

## Specific steps for your container / app server

- [Tomcat](tomcat.md)
- [JBoss](jboss.md)
- [WebLogic](weblogic.md)
- [WebSphere](websphere.md)
- [GlassFish](glassfish.md)
