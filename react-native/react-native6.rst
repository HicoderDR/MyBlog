.. post::Feb 24,2020
    :tags:react-native
    :category:react-native
    :author:HicoderDR

react-native初识：我使用的组件
#####################################################
首先，你必须先了解RN的基础组件

`这里是原生react-native的基础组件 <https://reactnative.cn/docs/components-and-apis/>`_

其次，可以在社区上找到很多更加好用的组件

我使用的组件
****************************
1.React Navigation(界面跳转、路由)
++++++++++++++++++++++++++++++++++++
`这里是Navigation官方文档 <https://reactnavigation.org/docs/en/getting-started.html>`_

首先修改App.js
--------------------
::

    import React from "react";
    import { Root } from "native-base";

    #引入 堆栈型导航器
    import { createStackNavigator, createAppContainer } from "react-navigation"; 
    #引入 我写的所有界面
    import Login from "./Login"
    import Mainpage from "./Mainpage"
    import Pay from "./Pay"
    
    #创建Route(路由)
    const AppNavigator = createStackNavigator(
        { 
            # 格式是 string:{screen: class}
            Login: {screen: Login},
            Main: {screen:Mainpage},
            Pay: {screen:Pay},
        },
        {
            initialRouteName: "Login"
        }
    ); 
    #将创建的Container直接设为app默认入口
    export default createAppContainer(AppNavigator);

    #如果希望在container外进行包装（例如包裹一个root）
    #const AppContainer = createAppContainer(AppNavigator);
    #export default class App extends React.Component {
    #   render() {
    #    return 
    #       <Root>
    #           <AppContainer />
    #       </Root>;
    #   }
    #}

然后在添加进路由的所有页面中使用
----------------------------------
::

    #最简单的--打开一个新界面，此时按回退键可以回到原界面
    this.props.navigation.navigate("Login")
    #销毁当前界面，替换成新界面
    this.props.navigation.replace("Login")
    #新界面压栈，与navigate区别是允许同一界面打开多个
    this.props.navigation.push("Login")
    #回退
    this.props.navigation.goBack()
    #如何携带参数
    this.props.navigation.navigate("ResT",{
        num:this.state.num,
    })

    #此外，还有更多堆栈操作，poptoTop()，reset()，dispatch()等

`这里是堆栈操作的官方API <https://reactnavigation.org/docs/en/stack-actions.html>`_

问:如果一个界面是子组件，不在路由中，但我希望它能使用navigation，怎么办？: 
    答：只需要将navigation传递给它就可以，例如
    ::

        <View>
            <History navigation={this.props.navigation}> </History>                
        </View>

`如何配置标题栏 <https://reactnavigation.org/docs/en/headers.html>`_

`如何与标题栏交互 <https://reactnavigation.org/docs/en/header-buttons.html>`_

2.Asyncstorage(本地存储)
+++++++++++++++++++++++++++++++++++++
`Github项目地址 <https://github.com/react-native-community/async-storage>`_

这个东西的使用其实是很坑的，没有达到我的期望

但能怎么办嘛，能用的就它一个了......

使用方法
::

    #首先要引入
    import AsyncStorage from '@react-native-community/async-storage';

    #定义一个storedata函数，可以传参
    storeData1 = async (Json) => {
      try {
        #这里变量名前必须加@，否则无效
        await AsyncStorage.setItem("@username", Json.data.userName);
      }
      catch(e){
      }
    }

    #定义函数的第二种写法
    async storeData2(Json){
      const first = ["@username", Json.data.userName]
      const second = ["@password", Json.data.password]
      #为什么我要用""+userID，因为后面必须是string，否则无效
      const third=["@userid",""+Json.data.userID]
      try {
        #多组数存储必须使用multiset，不能写三个await，否则无效
        await AsyncStorage.multiSet([first, second, third]);
        #同理，也可以存结构体，list，map
      }catch(e){
      }
    }

    #取值(只要你存的进去 就取得出来)
    getData = async () => {
      try {
        const _username = await AsyncStorage.getItem('@username')
        const _password = await AsyncStorage.getItem('@password')
        if(_username !== null&&_password!== null) {
          this.setState({
            Username: _username,
            Password: _password,
          });
        }
      } catch(e) {
      }
    }

3.async，await，promise
++++++++++++++++++++++++++++++++
既然上面写了async，就顺便介绍一下

asynchrony（异步），没错，就是令人头秃的异步函数。

”同步“：
    你去外地上学(人生地不熟)，突然生活费不够了；此时你决定打电话回家，通知家里转生活费过来，可是当你拨出电话时，对方一直处于待接听状态(即：打不通，联系不上)，为了拿到生活费，你就不停的oncall、等待，最终可能不能及时要到生活费，导致你今天要做的事都没有完成，而白白花掉了时间。

“异步”：
    在你打完电话发现没人接听时，猜想：对方可能在忙，暂时无法接听电话，所以你发了一条短信(或者语音留言，亦或是其他的方式)通知对方后便忙其他要紧的事了；这时你就不需要持续不断的拨打电话，还可以做其他事情；待一定时间后，对方看到你的留言便回复响应你，当然对方可能转钱也可能不转钱。但是整个一天下来，你还做了很多事情。 或者说你找室友临时借了一笔钱，又开始happy的上学时光了

**为什么取个值就要异步？**

因为主线程（UI线程）是不允许出现任何卡顿的，Android如果卡顿时间超过1s，程序就会直接被强制关闭。

那问题来了----我必须等到数据取完才能做下一步，必须要等，怎么办呢？
::

    #android JAVA 编程里的写法是
    int flag=0;
    getData();
    while(!flag){
        #让线程睡1ms，不写就等着爆炸把
        sleep(1);
    }

没错，就是这么真实，再看看RN
::
    
    #使用.then((parm)=>{   })的嵌套，以保证运行顺序
    read('1.txt').then((res)=>{
        return read('2.txt');  # 返回新的数据，然后输出
    }).then((res) => {
        return read('3.txt');
    }).then((res) => {
        console.log(res.toString());
    });

    #使用async+await
    #每当运行到await，就必须等待它的执行，可读性很强
    async function readByAsync(){
        let a1 = await read('1.txt');
        let a2 = await read('2.txt');
        let a3 = await read('3.txt');
        console.log(a1.toString());
    }

async的特点：
    - 语义化强

    - 里面的await只能在async函数中使用

    - await后面的语句可以是promise对象、数字、字符串等
    
    - async函数返回的是一个Promsie对象
    
    - await语句后的Promise对象变成reject状态时，那么整个async函数会中断，后面的程序不会继续执行

4.NativeBase
++++++++++++++++++++++++++++
这个就不多说了，真的是见到的最棒的第三方库了

`不管问几次 我都吹爆 NativeBase <https://docs.nativebase.io/>`_

5. react-native-vector-icons(精美图标库)
+++++++++++++++++++++++++++++++++++++++++++++
`npm官方文档 <https://www.npmjs.com/package/react-native-vector-icons>`_

如何使用
::

    import MaterialCommunityIcons from 'react-native-vector-icons/MaterialCommunityIcons';
    import FontAwesome from 'react-native-vector-icons/FontAwesome';

    #在render函数中想要使用的位置，添加对应的图片
    <FontAwesome
        name={'wpforms'}
        size={30}
        color={tintColor}
    />
    <MaterialCommunityIcons
        name={'face'}
        size={30}
        color={tintColor}
    />     

`在这里查找图片索引和对应的图标库 <https://oblador.github.io/react-native-vector-icons/>`_

6.CustomImage（图片预加载）
++++++++++++++++++++++++++++++++++
在图片未加载完成前使用loading.png

编写CustomImage.js
::

    import React, {Component} from 'react';
    import {Image, StyleSheet, View} from "react-native";
    import PropTypes from 'prop-types';

    export default class CustomImage extends Component {
        constructor(props) {
            super(props);
            this.state = {
                isLoadComplete: false,
                type: 0,//0,正常加载，1加载错误，
            }
        }
        static propTypes = {
            uri: PropTypes.any.isRequired,
            errImage: PropTypes.number,// 加载错误图片，可不传
            defaultImage:PropTypes.number,//预加载图片
            stylenew:PropTypes.style,
        }
        static  defaultProps = {
            #定义default和error
            defaultImage:require('../android/app/src/main/res/mipmap-hdpi/load.png'),
            errImage: require('../android/app/src/main/res/mipmap-hdpi/error.png'), //默认加载错误图片，可在此统一设置
        }

        render() {
            const {uri,defaultImage,errImage, style,stylenew} = this.props;
            let source = {uri};
            if (this.state.type === 1) {
                source = errImage;
            }
            return (
                <View style={[stylenew,style]}>
                    <Image
                        source={source}
                        style={[stylenew,{overflow: 'hidden', position: 'absolute',}, style]}
                        onError={(error) => {
                            this.setState({
                                type: 1,
                            })
                        }}
                        onLoadEnd={() => {
                            this.setState({
                                isLoadComplete: true,
                            })
                        }}
                    />
                    {this.state.isLoadComplete ?null: <Image style={[stylenew,style]} source={defaultImage}/> }
                </View>
            );
        }
    }

在需要使用的地方
::

    <CustomImage uri={item.uri} stylenew={styles.iconStyle}/>

7.react-native-image-crop-picker(相机和相册)
++++++++++++++++++++++++++++++++++++++++++++++++++
`这里是imagepicker官方文档 <https://github.com/ivpusic/react-native-image-crop-picker>`_

.. image:: ../_static/image-picker.png 

使用方法
::

    import {Button} from 'react-native-material-ui';
    import ImagePicker from 'react-native-image-crop-picker';

    #在任意组件的onpress事件中触发
    <Button raised primary text="拍照" style={{container:{borderRadius: 50, height:50, width:screenW*3/4},text:{fontSize:19}}} icon="camera-alt"
        onPress={()=> ImagePicker.openCamera({
            #可以设定裁剪和宽高
            cropping: cropping,
            width: 500,
            height: 500,
        }).then((image) => {
            #可以直接通过 image.path .width .height 来获得信息
            this.showtoast("服务器正在处理您的照片，请耐心等待","success")
        })}
    />

    <Button flat primary text="从相册选择" 
        style={{container:{height:50,width:screenW/3}}} 
        icon="camera"
        onPress={()=>ImagePicker.openPicker({
                #设定为可以选择多张
                multiple: true,
            }).then(images => {
                #如果要对images分别处理
                #for(var i in images){
                #   const path=images[i].path;     
                #}
                this.showtoast("服务器正在处理您的照片，请耐心等待","success")
            }
        )}
    />

8.react-native-share(分享组件)
+++++++++++++++++++++++++++++++++++++
`这里是Github网址 <https://github.com/songxiaoliang/react-native-share>`_

.. image:: ../_static/share.png

9.react-native-material-ui(UI组件库)
++++++++++++++++++++++++++++++++++++++++++
`这里是material-ui Github网址 <https://github.com/xotahal/react-native-material-ui>`_

.. image:: ../_static/material-ui.png 

10.react-native-image-zoom-viewer(图片查看组件)
++++++++++++++++++++++++++++++++++++++++++++++++++++
`这里是image-viewer Github网址 <https://github.com/magicwing/react-native-image-zoom-viewer>`_

.. image:: ../_static/image-zoom1.gif 

.. image:: ../_static/image-zoom2.gif 

.. _header-n0:

React-native扩展：优化的组件
============================

.. contents::
.. _header-n6:

11. `react-native-image-progress <https://github.com/oblador/react-native-image-progress>`__ ：图片加载中的进度动画
---------------------------------------------------------------------------------------------------------------

很简洁的文档，建议搭配下面的组件使用。

.. _header-n55:

12. `react-native-img-cache <https://github.com/wcandillon/react-native-img-cache>`__\ ：本地缓存的图片组件
-------------------------------------------------------------------------------------------------------

.. _header-n15:

CachedImage
~~~~~~~~~~~

.. _header-n39:

basic usage
^^^^^^^^^^^

.. code:: react

   import {CachedImage} from "react-native-img-cache";

   

   <CachedImage source={{ uri:"https://i.ytimg.com/vi/default.jpg" }} />

原来使用的 Image 标签渲染的图片，不会本地缓存，导致每次都需要加载一次。

换用缓存后效果有显著提升。

.. _header-n40:

costum usage
^^^^^^^^^^^^

.. code:: react

   <CustomCachedImage

     component={Image}

     source={{ uri: 'http://loremflickr.com/640/480/dog' }} 

     indicator={ProgressBar} 

     style={{

       width: 320, 

       height: 240, 

     }}/>

可以使用\ `react-native-image-progress <https://github.com/oblador/react-native-image-progress>`__\ 中的动画
**progressbar(进度条)\pie(饼图)\Circle(圆圈)**\ 来显示加载进度，好看但并不那么实用，可能会因为
Art 版本问题在IOS上报错。

.. _header-n45:

ImageCache
~~~~~~~~~~

.. _header-n25:

clear() ：清空
^^^^^^^^^^^^^^

Remove cache entries and all physical files.

.. code:: 

   ImageCache.get().clear();

.. _header-n28:

bust(uri) ：选择一个url从缓存中剔除
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``ImageCache`` can be used to bust an image from the local cache. This
removes the cache entry but it **does not remove any physical files**.

.. code:: 

   ImageCache.get().bust("https://i.ytimg.com/vi/yaqe1qesQ8c/maxresdefault.jpg");

.. _header-n31:

cancel(uri) ： 取消对某张图的下载
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It can also be used to cancel the download of an image. This can be very
useful when `scrolling through
images <https://medium.com/@wcandillon/image-pipeline-with-react-native-listview-b92d4768b17c>`__.

.. code:: 

   ImageCache.get().cancel("https://i.ytimg.com/vi/yaqe1qesQ8c/maxresdefault.jpg");

.. _header-n34:

on(uri, observer, immutable) 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``ImageCache`` class can register observers to the cache.

.. code:: 

   const immutable = true;

   const observer = (path: string) => {

       console.log(`path of the image in the cache: ${path}`);

   };

   ImageCache.get().on(uri, observer, immutable);

We use the observer pattern instead of a promise because a mutable image
might have different version with different paths in the cache.

.. _header-n62:

13. `react-native-dialog <https://github.com/mmazzarolo/react-native-dialog>`__\ ：弹出对话框
-----------------------------------------------------------------------------------------

良心组件，Android和IOS都有
