.. _header-n0:

IOS 内购---奔溃一条龙
=====================

上交App Store
的应用果然被退了回来，居然只出了两个问题，这是我始料未及的。

1. info.plist里面申请权限的内容没有具体说清楚。

2. 不允许直接在应用内使用其他付款方式或跳转支付，而要使用 IAP（App Store
   内购）。

头好疼....

.. _header-n22:

内购标准逻辑（官方）
--------------------

先贴一张系统顺序图（笑）

.. figure:: ../_static/react-native-iap.svg

1.  程序向服务器发送请求，获得一份产品列表。

2.  服务器返回包含产品标识符的列表。

3.  程序向App Store发送请求，得到产品的信息。

4.  App Store返回产品信息。

5.  程序把返回的产品信息显示给用户（App的store界面）

6.  用户选择某个产品

7.  程序向App Store发送支付请求

8.  App Store处理支付请求并返回交易完成信息。

9.  程序从信息中获得数据，并发送至服务器。

10. 服务器纪录数据，并进行审(我们的)查。

11. 服务器将数据发给App Store来验证该交易的有效性。

12. App Store对收到的数据进行解析，返回该数据和说明其是否有效的标识。

13. 服务器读取返回的数据，确定用户购买的内容。

14. 服务器将购买的内容传递给程序。

**看完我头就秃了 什么??? 我千疮百孔的支付逻辑要被迫重生了嘛？**

.. _header-n61:

协议、银行卡信息的填写
----------------------

https://www.jianshu.com/p/4f5f0b45b083

**当然，是交给甲方或者项目经理做了**

.. _header-n71:

配置内购产品、测试账号
----------------------

https://www.jianshu.com/p/89acda082b07

https://www.jianshu.com/p/d65c7c0027b8

当然，这个很简单，仅仅是在Apple的服务器上加一个产品列表和一个沙箱账号

.. _header-n80:

修改项目代码
------------

.. _header-n86:

RN端
~~~~

`react-native-in-app-utils <https://github.com/chirag04/react-native-in-app-utils>`__
：前端调用时使用的组件（仅有IOS）

`react-native-iap <https://github.com/dooboolab/react-native-iap>`__
：Android和IOS都可以使用的内购组件

熟读完上述两个库之后，你就无敌了（整个支付流程烂熟于心）。

.. _header-n120:

服务端
~~~~~~

https://blog.csdn.net/endtheme_xin/article/details/83087223

这是Springboot 也就是 JAVA端实现IAP的一份代码（令人头秃）

https://blog.csdn.net/kaikai136412162/article/details/79670622

Python的一份，但是仅仅是测试，不是很全。

.. _header-n132:

真机调试
--------

内购支付在模拟器上是永远失败的。

所以只能真机调试（谁买得起IOS呢）

所以，我选择\ `云真机 <https://www.testin.cn/>`__ 。
