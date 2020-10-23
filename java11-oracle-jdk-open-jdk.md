## Java11时代Oracle JDK与Open JDK区别
### Java11时代
　　Java 的版本发布周期变更为每六个月一次 ， 每半年发布一个大版本，每个季度发布一个中间特性版本，Java 9 和 Java 10 这两个被称为“功能性的版本”，两者均只提供半年的技术支持，Java 11 不仅提供了长期支持服务，还将作为 Java 平台的参考实现。Oracle 直到2023年9月都会为 Java 11 提供技术支持，而补丁和安全警告等扩展支持将持续到2026年。所以，Java11 必将是下一代长期使用的版本。

### Open JDK　　
　　在Java历史上，openjdk是jdk的开放原始码版本，以GPL协议的形式放出。在JDK7的时候，openjdk已经成为jdk7的主干开发，sun jdk7是在openjdk7的基础上发布的，其大部分原始码都相同，只有少部分原始码被替换掉。使用JRL(JavaResearch License，Java研究授权协议)发布。

### Java11时代，Oracle JDK与Open JDK
1、授权协议的不同： openjdk采用GPL V2协议放出，而JDK则采用JRL放出。两者协议虽然都是开放源代码的，但是在使用上的不同在于GPL V2允许在商业上使用，而JRL只允许个人研究使用。 Oracle JDK源代码包括“ORACLE PROPRIETARY / CONFIDENTIAL。使用受许可条款约束。

2、OpenJDK不包含Deployment（部署）功能： 部署的功能包括：Browser Plugin、Java Web Start、以及Java控制面板，这些功能在Openjdk中是找不到的。 

3、openjdk只包含最精简的JDK：Oracle JDK提供“JDK”和“JRE”， OpenJDK仅提供 ”JDK“。Oracle JDK提供“安装程序”（msi，rpm，deb等），将JDK二进制文件放在系统中，还包含更新规则，在某些情况下还处理一些常见配置，如设置公共环境变量（例如，JAVA_HOME in Windows）并建立文件关联（例如，使用java来启动.jar文件）。 OpenJDK仅作为压缩存档（tar.gz或.zip）提供。 OpenJDK不包含其他的软件包，比如Rhino Java DB JAXP……，并且可以分离的软件包也都是尽量的分离，但是这大多数都是自由软件，你可以自己下载加入。 

4、OpenJDK源代码不完整：Oracle JDK二进制文件包括未添加到OpenJDK二进制文件的API，如javafx，资源管理和（JDK 11之前的更改）JFR API。在采用GPL协议的Openjdk中，sun jdk的一部分源代码因为产权的问题无法开放openjdk使用，其中最主要的部份就是JMX中的可选元件SNMP部份的代码。因此这些不能开放的源代码将它作成plug，以供OpenJDK编译时使用，你也可以选择不要使用plug。而Icedtea则为这些不完整的部分开发了相同功能的源代码(OpenJDK6)，促使OpenJDK更加完整。 

5、部分源代码用开源代码替换： 由于产权的问题，很多产权不是SUN的源代码被替换成一些功能相同的开源代码，比如说字体栅格化引擎，使用Free Type代替。 

6、OpenJDK不支持启动参数：如果使用-XX：+ UnlockCommercialFeatures标志，OpenJDK将（继续）抛出错误并停止。 Oracle JDK不再需要该标志，并将打印警告，但如果使用则继续执行。

7、不能使用Java商标：Oracle JDK有Java杯和蒸汽图标，OpenJDK有Duke图标。java -version的输出会有所不同。 Oracle JDK包含LTS。 OpenJDK（由Oracle生成）不包括Oracle特定的LTS标识符。
