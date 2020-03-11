.. post::Oct 11,2019
    :tags:react-native
    :category:react-native
    :author:HicoderDR

react-native进阶:秒嘀云+Nativebase实现短信验证码登陆
#############################################################
秒嘀云平台的使用和服务端的配置
***************************************
首先要使用短信验证码服务，你需要通过企业认证（法人和执照），
否则只能老老实实写个账密登陆自high.

`秒嘀云官网 <http://www.miaodiyun.com/>`_

第一步:
    注册并登陆，且通过如下的企业认证(审核仅需十分钟不到)

.. image:: ../_static/mdyrz.png

第二步：
    找到短信配置页面，里面有必要的信息，并在IP栏把你服务器的公网IP加上
    ，然后下载Demo，Demo有Java和Python两个版本，以下我使用Java后台。

.. image:: ../_static/dxpz.png

第三步：
    找到模板管理页面，创建自己的模板，我的模板中只有一个变量{1},
    创建完后会看到自己的模板ID，记住即可

.. image:: ../_static/mdymb.png

第四步:
    打开Demo，发现只有三个文件，一个Test，一个Config，一个httpUtil，
    首先将Config复制到自己的项目里，里面的信息换成自己的，
    然后将httpUtil原样复制到项目里
    
.. image:: ../_static/mdyconfig.png

最后，我们看一下Test里面的内容

.. image:: ../_static/mdytest.png

简单易懂，to即发送至的手机号，param即模板中的变量，templateid即模板ID

第五步：
    现在开始写自己的后台,首先申明一个接口(我使用的是restful API,这不重要,大家只要关注核心代码即可)

::
    
    @PostMapping("/api/user/postsms")
    public String postsms(@RequestParam String phone){
        try{
            #生成一个六位随机数验证码
            String Code = String.valueOf(new Random().nextInt(899999) + 100000);
            #下面照抄Demo
            StringBuilder sb = new StringBuilder();
            sb.append("accountSid").append("=").append(MdyConfig.ACCOUNT_SID);
            sb.append("&to").append("=").append(phone);
            sb.append("&templateid").append("=").append("236029");
            #模板中的变量，如果有多个，用逗号分隔
            sb.append("&param").append("=").append(Code);
            String body = sb.toString() + HttpUtil.createCommonParam(MdyConfig.ACCOUNT_SID, MdyConfig.AUTH_TOKEN);
            #请求的结果可以返回或处理，这里我不多写
            String result = HttpUtil.post(MdyConfig.BASE_URL, body);
            #返回发送的验证码给前端
            return Code;
        }
        catch (Exception  e){
            return "发送验证码失败";
        }
    }

至此秒嘀云服务就成功接入了

Nativebase写前端并调用验证码接口
***************************************
为了方便理解，我还是将代码拆分成界面实现和逻辑两个部分

本部分的实现会涉及
`NativeBase <https://docs.nativebase.io/>`_
和
`React navigation <https://reactnavigation.org/docs/en/getting-started.html>`_
，必须有基本了解。

界面实现
++++++++++++++++++++++++++++++++
首先看验证码登陆界面

.. image:: ../_static/smslogin.png

内部结构是 两个Input输入框，一个验证码button，一个登陆button

::

    import React, { Component } from 'react'
    import {View,ImageBackground,Dimensions} from "react-native"
    import {
        Text,Header,Container,
        Content, Form, Item,
        Input, Label, Left, 
        Body, Right,Title,
        Icon,Button,Toast,
        Row,Col,Grid,
    } from "native-base";
    #引入倒计时文本
    import CountDownText from "./CountDownText"

    
    const screenW = Dimensions.get('window').width;
    const screenH = Dimensions.get('window').height;

    export default class MSLogin extends Component{
        constructor(props){
            super(props);
            this.state={
                phone:"",           #两个输入框的内容
                smscode:"",
                getting:false,      #发送验证码
                phonelock:false,    #使手机号输入框不可更改
                correct:"",         #存储服务器返回的正确验证码
            }
        }
        static navigationOptions = {
            header:null,
        }
        render(){
            return(
                <Container>
                #设置背景图
                <ImageBackground 
                    style={{ width:screenW,height:screenH }}
                    source={require("./android/app/src/main/res/mipmap-xxxhdpi/newback.png")}
                >
                <View style={{flex:5}}>

                    # 使用沉浸式Header头，融入背景
                    <Header transparent style={{marginTop:10}}>
                        <Left>
                            <Button transparent
                                onPress={()=>{this.props.navigation.replace("Login");}}
                            >
                                <Icon name='arrow-back' style={{color:"white"}} />
                            </Button>
                        </Left>
                        <Body>
                            <Title>短信验证登陆</Title>
                        </Body>
                        <Right></Right>
                    </Header>

                    <Form style={{margin:15,marginTop:screenH/20}}>
                        
                        <Item rounded style={{margin:5}}>
                            <Icon active  type="AntDesign" name="mobile1" style={{color:"white"}}/>
                            
                            #手机号输入框
                            <Input 
                                style={{color:"white"}}          #输入字样式
                                placeholder='请输入您的手机号'    #提示字
                                placeholderTextColor="white"     #提示字样式
                                autoFocus={true}                 #进入页面后自动获取焦点
                                blurOnSubmit={true}              #按下键盘 确定/提交 键后失焦
                                keyboardType="numeric"           #调用手机纯数字键盘
                                maxLength={11}                   #最大长度---手机号11位
                                editable={!this.state.phonelock} #发送验证码后将手机号一栏锁定
                                value={this.state.phone}         
                                onChangeText={(txt)=>{
                                    this.setState({phone:txt})
                                }}
                            />
                        </Item>
                        
                        <Item rounded style={{margin:5}}>
                            <Icon active  type="MaterialIcons" name="sms" style={{color:"white"}}/>
                            
                            #验证码输入框
                            <Input placeholder='请输入短信验证码'
                                placeholderTextColor="white"
                                style={{color:"white"}}
                                keyboardType="numeric"
                                maxLength={6}
                                value={this.state.smscode}
                                onChangeText={(txt)=>{
                                    this.setState({smscode:txt})
                                }}
                            />

                            #使用三目运算符 ？： 发送后变为倒计时文本
                            {this.state.getting ?
                            
                            #申明按钮为 disabled
                            <Button disabled rounded small light style={{opacity:0.6,marginRight:5}}>
                                <CountDownText                                          # 倒计时
                                    style={{color:"#3498DB"}}
                                    countType='seconds'                                 # 计时类型：seconds / date
                                    auto={true}                                         # 自动开始
                                    afterEnd={() => {this.setState({getting:false})}}   # 结束回调
                                    timeLeft={60}                                       # 时间起点
                                    step={-1}                                           # 计时步长，以秒为单位
                                    startText='获取验证码'                               # 开始的文本
                                    endText='获取验证码'                                 # 结束的文本
                                    intervalText={(sec) => sec + '秒重新获取'}           # 定时的文本回调
                                />
                            </Button>
                            
                            :

                            #触发发送验证码的按钮
                            <Button rounded small light
                                style={{opacity:0.6,marginRight:5}}
                                onPress={()=>{
                                    this.sendsms()
                                }}
                            >
                                <Text style={{color:"#3498DB"}}>获取验证码</Text>
                            </Button>
                            }
                        </Item>
                    </Form>

                    #登陆按钮
                    <Button  block light style={{marginTop:30,marginRight:15,marginLeft:15,opacity:0.7}}
                        onPress={this.login}
                    >
                        <Text style={{fontSize:18,color:"#3498DB"}}>登 录</Text>
                    </Button>
                    
                </View>
                <View style={{flex:2}}></View>
                </ImageBackground>
                </Container>
            );
        }
    } 

CountDownText我是借鉴的别人的代码，挂出
`github地址 <https://github.com/jsonz1993/wheel/tree/master/react-native-countdown>`_
，我将他的两个js文件复制到项目中，并注释掉此处代码，
使得他不会因为setstate而不停地回退刷新时间

.. image:: ../_static/countdowntext.png

逻辑实现
++++++++++++++++++++++++++++++++

:: 

    #LoadingView有多种实现方式，我的实现方式也会写在下方
    import MyLoadingUtil from "./myclass/MyLoadingUtil"
    import AsyncStorage from '@react-native-community/async-storage'

    #服务器地址
    const server="http://xx.xx.xx.xx:port/";

    export default class MSLogin extends Component{

        #发送验证码
        sendsms(){
            #获取当前值
            const phone=this.state.phone;
            #判断输入合法
            if(phone.length===11) {
                #发送http请求
                fetch(server+'api/user/postsms?phone='+phone, {
                    method: 'POST',
                }).then((response)=>{
                    return response.text();
                }).then((response) =>{
                    #由于我的后台直接返回验证码字符串
                    #所以只需.text()后，即可将其保存在correct中
                    this.setState({
                        correct:response,
                        getting:true,
                        phonelock:true
                    })
                    this.showtoast("验证码已发送","success")
                }).catch(e=>{
                    this.showtoast("服务器无响应","warning")
                })
            }
            else{
                this.showtoast("请检查您的输入","warning")
            }
        }

        #登陆 (可以看到两种不同写法，注意在onpress调用的时候写法也不同)
        login=()=>{
            const phone=this.state.phone;
            const smscode=this.state.smscode;
            #判断是否正确
            if(phone.length===11 && smscode===this.state.correct &&smscode.length==6){
                #此时已经可以判断成功登陆，进行跳转，不做以下步骤
                
                #获取用户信息并将其保存，以便自动登陆
                MyLoadingUtil.showLoading();
                fetch(server+'api/user/getuserinfo?username='+phone, {
                    method: 'GET',
                })
                .then((response) => response.json())
                .then((Json)=>{
                    this.storeData(Json);
                    MyLoadingUtil.dismissLoading();
                }).then(()=>{
                    this.showtoast("欢迎使用！","success");
                    this.props.navigation.replace("Main");
                }).catch((e) => {
                    this.showtoast("网络连接失败","warning");
                    MyLoadingUtil.dismissLoading();
                });
            }else{
                this.showtoast("请检查您的输入","warning");
            }
        }
        render(){
            #..................
        }
        #重写showtoast方法
        showtoast(text,type){
            Toast.show({
                text: text,
                buttonText: "好的",
                type: type
            });
        }
        #本地存储
        storeData = async(Json) => {
            const first = ["@username", Json.data.userName]
            const second = ["@password", Json.data.password]
            const third=["@userid",""+Json.data.userID]
            try {
            await AsyncStorage.multiSet([first, second, third]);
            }
            catch(e){}
        }
    } 

`我的LoadingView实现方法来源 <https://www.jianshu.com/p/89aed8ec4cc2>`_