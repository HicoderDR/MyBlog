.. post::Oct 5,2019
    :tags:react-native
    :category:react-native
    :author:HicoderDR

react-native实战:轮播引导和登陆注册
#################################################

react-native-swiper实现轮播图
***************************************
本部分的实现会涉及
`AsyncStorage <https://hicoderdr.github.io/react-native6/#asyncstorage>`_
、
`React navigation <https://reactnavigation.org/docs/en/getting-started.html>`_
和
`react-native-swiper <https://github.com/liyinglihuannan/react-native-swiper>`_
，必须有基本了解。

先看看效果
++++++++++++++++++++++++++++++++
.. image:: ../_static/swiper.gif

swiper部分的代码
++++++++++++++++++++++++++++++++
::

    import React, { Component } from 'react';
    import {
        View,
        ImageBackground,
        TouchableOpacity
    } from 'react-native'
    #引入Swiper
    import Swiper from 'react-native-swiper';

    export default class CriImage extends Component {
        static navigationOptions = {
            #去除React-navigation的默认header
            header: null,
        };
        render() {
            return (
                <Swiper 
                    loop={false}            #设定不循环
                    showsPagination={false} #是否显示默认分页的小圆点
                    autoplay={true}         #设定自动延时播放
                    style={{ flex: 1 }}
                >
                    <View style={{ flex: 1 }}>
                        #require获取本地图片
                        <ImageBackground style={{ flex: 1 }} source={require("./android/app/src/main/res/mipmap-hdpi/cri1.png")}>
                            <View style={{ flex: 1, justifyContent: 'flex-end', alignItems: 'flex-end' }}>
                            </View>
                        </ImageBackground>
                    </View>
                    <View style={{ flex: 1 }}>
                        <ImageBackground style={{ flex: 1 }} source={require("./android/app/src/main/res/mipmap-hdpi/cri2.png")}>
                            <View style={{ flex: 1, justifyContent: 'flex-end', alignItems: 'flex-end' }}>
                            </View>
                        </ImageBackground>
                    </View>

                    #最后一页我设定为 点击屏幕即跳转 所以套用TouchableOpacity
                    #也可以添加按钮跳转、或使用SetTimeout延时跳转
                    <View style={{ flex: 1 }}>
                        <ImageBackground style={{ flex: 1 }} source={require("./android/app/src/main/res/mipmap-hdpi/cri3.png")}>
                            <TouchableOpacity activeOpacity={0.8} 
                                onPress={()=>{
                                    #跳转至登陆界面
                                    this.props.navigation.replace("Login")
                                }}
                                style={{ flex: 1, justifyContent: 'flex-end', alignItems: 'flex-end' }}
                            ></TouchableOpacity>
                        </ImageBackground>
                    </View>
                </Swiper>
            );
        }
    }

设定对于老用户，不播放轮播图
++++++++++++++++++++++++++++++++
::

    #引入本地存储
    import AsyncStorage from '@react-native-community/async-storage';
    #首先保证用户在使用过APP后，本地存储了信息
    export default class CriImage extends Component {
        #.......
        
        #页面render前判断并跳转
        componentWillMount(){
            this.getData();
        }
        getData = async () => {
            try {
                const _username = await AsyncStorage.getItem('@username')
                if(_username!==null) this.props.navigation.replace("Login")
            } catch(e) {
            # error reading value
            }
        };
        #.......
    }

登陆注册界面的简单实现
*******************************************
我将代码拆解成三部分：UI、服务器请求和本地存储

先看看效果
++++++++++++++++++++++++++++++++
.. image:: ../_static/login.gif

实现登陆的UI代码
++++++++++++++++++++++++++++++++
::

    import React, { Component } from 'react';
    import { Button } from 'react-native-material-ui';
    import {
    json,
    Text,
    TextInput,
    View,
    StyleSheet,
    Image,
    Alert,
    Dimensions,
    } from 'react-native'
    import { Toast } from "native-base";

    const screenW = Dimensions.get('window').width;

    export default class Login extends Component{
        constructor (props) {
            super (props)
            this.state = {
                Username:"",
                Password:"",
                active:"false",
                showToast: false,
            }
        }
        static propTypes = {
            name1: String,
            txtHide1: String,
            ispassword1: Boolean,
            name2: String,
            txtHide2: String,
            ispassword2: Boolean,
            resultjson:json,
        }
        #设定props初值
        static defaultProps = {
            name1: '账号',
            txtHide1: '请输入您的用户名',
            ispassword1: false,
            name2: '密码',
            txtHide2: '请输入密码',
            ispassword2: true,
        }

        static navigationOptions = {
            header:null,
        };
        render(){
            var {  name1, txtHide1, ispassword1,name2, txtHide2, ispassword2 } = this.props
            return(
                #使用flexbox布局的写法
                <View style={{flex: 2}}>
                    #空白的占位View
                    <View style={{flex:2}}></View>

                    #本地人像图片 
                    <View style={{height:50,flexDirection:'row',justifyContent:'center'}}>
                        <Image
                            style={{width: 100, height: 100,borderRadius:50}}
                            source={require("./android/app/src/main/res/mipmap-hdpi/personimg.jpg")}
                        />
                    </View>

                    <View style={{flex:1}}></View>

                    #用户名输入框
                    <View style={{flex: 6, justifyContent: 'space-evenly',}}>
                        <View style={{ height: 50, flexDirection: 'row', justifyContent: 'center' }}>
                            <View style={styles.container}>
                                <View style={styles.txtBorder}>
                                    <Text style={styles.txtName}>{name1}</Text>
                                    <TextInput
                                        underlineColorAndroid = {'transparent'}
                                        style={styles.textInput}    
                                        multiline={false}           #禁止多行文本
                                        placeholder={txtHide1}      #框内未空白时的灰字提示
                                        password={ispassword1}      #调用password属性，true则输入后默认隐藏
                                        #实时回调 更新state
                                        onChangeText={(text) => {   
                                            this.setState({
                                                Username: text
                                            })
                                        }
                                        #渲染时默认填state内的值，如果不写，每次render就会刷新为空
                                        value={this.state.Username}
                                    />
                                </View>
                            </View>
                        </View>

                        #下一个框依葫芦画瓢
                        <View style={{height: 50,  flexDirection: 'row', justifyContent: 'center'}}>
                            <View style={styles.container}>
                                <View style={styles.txtBorder}>
                                    <Text style={styles.txtName}>{name2}</Text>
                                    <TextInput
                                        underlineColorAndroid = {'transparent'}
                                        style={styles.password}
                                        multiline={false}
                                        placeholder={txtHide2}
                                        password={ispassword2} 
                                        secureTextEntry={true}
                                        onChangeText={(text) => {
                                            this.setState({
                                                Password: text
                                            })
                                            console.log(text);
                                        }}
                                        value={this.state.Password}
                                    />
                                </View>
                            </View>
                        </View>

                        #登陆按钮
                        <View style={{height: 50,  flexDirection: 'row', justifyContent: 'center', alignItems: 'stretch'}}>
                            <Button raised primary 
                                text="登陆 / 注册" 
                                style={{container:{borderRadius: 50, height:50,width:320},text:{fontSize:19}}}
                                #设置点击事件
                                onPress = {()=>this._Check()}
                            />
                        </View>
                    </View>

                    <View style={{flex: 2,height: 50 }} ></View>
                </View>
            );
        }
        #调用底部Toast函数 (更多用法参考 native-base)
        showtoast(text,type){
            Toast.show({
                text: text,
                buttonText: "好的",
                type: type
            });
        }
        const styles = StyleSheet.create({
            container: {
                flex: 1,
                alignItems: 'center',
                flexDirection: 'row'
            },
            txtBorder: {
                height: 50,
                flex: 1,
                borderWidth: 1,
                borderColor: '#51A7F9',
                marginLeft: 40,
                marginRight: 40,
                borderRadius: 25,
                flexDirection: 'row',
                justifyContent:'center',
            },
            txtName: {
                height: 20,
                width: 40,
                marginLeft: 20,
                fontSize: 15,
                marginTop: 15,
                color: '#51A7F9',
                marginRight: 10,
                fontSize: 14
            },
            textInput: {
                height: 50,
                width: screenW-180
            },
            password: {
                height: 50,
                width: screenW-180
            },
            button: {
                alignItems: 'center',
                backgroundColor: '#DDDDDD',
                padding: 10,
                borderRadius:50,
                flexDirection: 'row',
            },
        })
    }

实现登陆的服务器代码
++++++++++++++++++++++++++++++++
如果没有后台服务器，也可以在本机简单逻辑判断。

`React Native发送http请求的方式 <https://reactnative.cn/docs/network/>`_
:Fetch、ajax或第三方的axios等，以下我使用RN官方的Fetch为例。

我的逻辑：
    我为了方便，登陆和注册使用同一个按钮，首先询问该用户名的用户是否存在，不存在即注册，存在即判断密码是否正确。

::

    #服务器地址
    const server="http://xxx.xxx.xxx.xxx:port/"

    export default class Login extends Component{
        #为了方便定义了取值函数
        getUsername(){
            return this.state.Username;
        }
        getPassword(){
            return this.state.Password;
        }

        #检测是否存在账户
        _Check(){
            #判断是否不合法
            if(this.getUsername()==""||this.getPassword()==""){
                this.showtoast("请检查用户名和密码","warning");
                return;
            }
            #发送请求询问该用户名
            fetch(server+'api/user/searchbyname?username='+this.getUsername(), {
                method: 'GET',
            })
            .then((response) => response.json())
            .then((Json) => {
                #根据服务器response值判断为登陆还是注册
                if(Json.data == "true")   this._Login();
                else                      this._adduser();
            })
            .catch((error) => {
                #显示黄色Toast
                this.showtoast("网络连接失败","warning")
            });
        };

        _adduser(){
            fetch(server+'api/user/adduser?username='+this.getUsername()+'&nickname='+this.getUsername()+'&password='+this.getPassword(), {
                method: 'POST',
            })
            .then((response) => response.json())
            .then((Json) => {
                #将账密存在本地，下次自动登陆
                this.storeData(Json);
            })
            #success显示绿色Toast
            .then(this.showtoast("注册成功，祝您愉快","success"))
            .then(this.props.navigation.replace("Main"))
        };

        _Login(){
            #设置一个flag标记，不过这是很不优雅的写法（不要学）
            var flag=1;
            fetch(server+'api/user/getuserinfo?username='+this.getUsername(), {
                method: 'GET',
            })
            .then((response) => response.json())
            .then((Json) => {
                if(Json.data.password==this.getPassword()){
                    return Json;
                }
                else{
                    flag=0;
                    this.showtoast("密码错误或用户名已被占用","warning");
                }
            }).then((Json)=>{
                if(flag==0) return;
                #将账密存在本地，下次自动登陆
                this.storeData(Json);
                this.showtoast("欢迎使用！","success")
            }).then(()=>{
                if(flag==0) return;
                #跳转至Main页面
                this.props.navigation.replace("Main")
            }).catch((error) => {
                this.showtoast("网络连接失败","warning")
            });
        }
    }

老用户自动登陆
++++++++++++++++++++++++++++++++
::

    #引入AsyncStorage
    import AsyncStorage from '@react-native-community/async-storage';

    export default class Login extends Component{
        #.....

        #组件生命周期自动调用
        componentWillMount(){
            this.getData();
        }
        #存储
        storeData = async (Json) => {
            const first = ["@username", Json.data.userName]
            const second = ["@password", Json.data.password]
            #因为userID是int不是String 所以特别处理一下
            const third=["@userid",""+Json.data.userID]
            try {
                #将值存在本地
                await AsyncStorage.multiSet([first, second, third]);
            }
            catch(e){
            }
        }
        #获取
        getData = async () => {
            try {
                const _username = await AsyncStorage.getItem('@username')
                const _password = await AsyncStorage.getItem('@password')
                if(_username !== null&&_password!== null) {
                    #将username和password自动填入
                    this.setState({
                        Username: _username,
                        Password: _password,
                    });
                    #调用登陆函数
                    this._Login();
                }
            } catch(e) {
                # error value
            }
        }
    }