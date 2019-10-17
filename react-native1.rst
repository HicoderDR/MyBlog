.. post::Sep 06 , 2019
   :tags: react-native
   :category: react-native
   :author: HicoderDR

react-native考察：对比Flutter
###########################################

`react-native <https://reactnative.cn/>`_ (RN),多平台开发利器。
   
如果你是FaceBook的忠实粉丝,想必不会陌生。
    
接触react-native的原因
***********************
    最近接下了同济大学的钢筋计数项目，要求Android、IOS双平台开发。
    之前从未接触过IOS开发，所以略有慌张。好在之前听说了Flutter的大名，开会的时候才大言不惭。
            
    在此必须隆重介绍一下 `Flutter <https://flutterchina.club/>`_ .
    谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。
            
    听起来很棒对不对？
    
    但是我最终放弃了它选择了RN，这就要说回最初的话题了。

跨平台开发移动应用的思考
************************

第一种办法：原生开发
    IOS 采用Xcode，基于XML渲染视图，支持开发人员使用 C、 C++、Objective C、 AppleScript 和 Java。
    
    Android 采用Android Studio，基于XML渲染视图，支持开发人员使用Java、Kotlin。
        
    单独使用其中任何一种都很赞，文档齐全，教程很棒，可调用的第三方库也很多。

    但是如果要双平台开发，无疑是两倍的代码量，几乎无法代码重用。

第二种办法：APP内嵌套WebView
    主要借助Web开发，PWA对移动手机触发事件的相应，完成一个可实现类似APP功能的网页。

    虽然十分逼真，而且网页天然适配Android和Ios双系统，但这无疑是将APP的逼格降了一个档次。

第三种办法：Flutter或者React-native
    Flutter：
        Flutter将Android和Ios的原生同类组件封装在一起，通过os.system判断进行自适应。
        真正做到了“写一份代码，适配两个系统的伟大工程”。

        但Flutter采用的Dart语言给我一种介于JavaScript和Java之间的错觉，
        还有它较为年轻的身形，并不完善的文档，人数待扩张的社区，致使我放弃了它。

    React-native： 
        RN适应新的ES6标准，同样对原生组件进行了封装，
        虽然很多组件不能在双系统间通用，但总能找到替代品，
        RN保留了原生Android和Ios的文件结构，需要分别配置，
        在节省代码量上不如Flutter。
        
        **相比较于Flutter来说，各种Bug都有更为完善的文档和解决方案。**
        
        **RN使用CSS和JSX，给了开发者一种正在写网页的熟悉感，上手更快。**


开学季
