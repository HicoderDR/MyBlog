.. post::Oct 24,2019
    :tags:springboot
    :category:springboot
    :author:HicoderDR

Springboot初识:了解Spring
#############################################################
`spring官网地址 <https://spring.io/>`_

创建团队：Pivotal

Spring
*****************
Spring的思想:
    Spring框架为开发提供了一系列的解决方案，比如利用控制反转的核心特性，
    并通过
    **依赖注入**
    实现控制反转来实现管理对象生命周期容器化，利用面向切面编程进行声明式的事务管理，
    整合多种持久化技术管理数据访问，提供大量优秀的Web框架方便开发等等。
    Spring框架具有控制反转（IOC）特性，IOC旨在方便项目维护和测试，
    它提供了一种通过Java的反射机制对Java对象进行统一的配置和管理的方法。
    Spring框架利用容器管理对象的生命周期，容器可以通过扫描XML文件或类上特定Java注解来配置对象，
    开发者可以通过依赖查找或依赖注入来获得对象。

Spring框架：
    * 框架是指 **能完成一定功能的半成品** 
    * Spring 全称 **Spring Framework** ，包含了 **Springboot（脚手架）** ，**SpringMVC（基于MVC设计的web框架）** ，**Spring Cloud（构建云服务的框架）** ，**Spring Security（安全访问控制）** 等等……


Spring的特点:
    #. 可以创建独立的Spring应用程序，并且基于其Maven或Gradle插件，可以创建可执行的JARs和WARs
    #. 内嵌Tomcat或Jetty等Servlet容器
    #. 提供自动配置的“starter”项目对象模型（POMS）以简化Maven配置
    #. 尽可能自动配置Spring容器
    #. 提供准备好的特性，如指标、健康检查和外部化配置
    #. 绝对没有代码生成，不需要XML配置


SpringBoot
*****************
SpringBoot仅仅是一个脚手架，能帮助你快速地构建和部署项目，并不限制模型和设计模式的使用。

使用Spring Boot构建的项目，都可以称为使用了springboot框架

Springboot应用支持:
    * 前端常使用模板引擎，主要有FreeMarker和Thymeleaf，它们都是用Java语言编写的，渲染模板并输出相应文本，使得界面的设计与应用的逻辑分离，同时前端开发还会使用到Bootstrap、AngularJS、JQuery等
    * 在浏览器的数据传输格式上采用 **Json** ，非xml，同时提供 **RESTful API**
    * SpringMVC框架用于数据到达服务器后处理请求
    * 到数据访问层主要有Hibernate、MyBatis、JPA等持久层框架
    * 数据库常用 **MySQL**
    * 开发工具推荐 `IntelliJ IDEA <http://www.jetbrains.com/idea/>`_
