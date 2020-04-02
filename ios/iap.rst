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

这里我使用的是 React-native-iap

首先
::

   yarn install react-native-iap
   cd ios && pod install && cd ..   # CocoaPods on iOS needs this extra step

完成安装后，首先在应用的最外层（这里是App类），设置支付监听。
::
   
   import React from "react";
   import { View ,Alert} from 'react-native';

   import RNIap, {
   purchaseErrorListener,
   purchaseUpdatedListener,
   type ProductPurchase,
   type PurchaseError
   } from 'react-native-iap';

   import server from './server'
   import AsyncStorage from '@react-native-community/async-storage';

   export default class App extends React.Component {
      
      #从这里开始
      purchaseUpdateSubscription = null
      purchaseErrorSubscription = null
      
      componentWillUnmount() {
         if (this.purchaseUpdateSubscription) {
            this.purchaseUpdateSubscription.remove();
            this.purchaseUpdateSubscription = null;
         }
         if (this.purchaseErrorSubscription) {
               this.purchaseErrorSubscription.remove();
               this.purchaseErrorSubscription = null;
         }
      }
      async componentDidMount(){
         this.purchaseUpdateSubscription = purchaseUpdatedListener((purchase: InAppPurchase | SubscriptionPurchase | ProductPurchase ) => {
            console.log('purchaseUpdatedListener', purchase);

            #支付流程为客户端向Apple发送支付请求，支付完成后返回到客户端
            #purchase中即为返回的订单对象，接下来进行自己服务器的购买逻辑

            #receipt收据 （验证就靠它） 划重点！！
            var receipt = purchase.transactionReceipt;
            AsyncStorage.getItem('@username').then(username =>{
               if (receipt && username!='') {
                  const formdata = new FormData();
      
                  formdata.append("username",username);
                  formdata.append("receipt",receipt);
                  
                  fetch(server+"buyios",{
                     method: 'POST',
                     body: formdata,
                     headers: {
                        "Content-Type": "multipart/form-data"
                     }
                  }).then(resp => resp.text())
                  .then(resp =>{
                     if(resp === "success"){
                        Alert.alert("到账通知","你的充值已到账");
                        
                        #完成充值后 finishTransaction 通知苹果服务器支付完成（第二个参数为 是否为沙盒环境）
                        RNIap.finishTransaction(purchase, false);
                        try{
                           RNIap.finishTransaction(purchase, true);
                        }catch(e){
                        }
                     }
                  })
               }
            })
         });

         this.purchaseErrorSubscription = purchaseErrorListener((error: PurchaseError) => {
            console.warn('purchaseErrorListener', error);
         });
      }
      render() {
         return <Root>
            <View style={{flex : 1,}}>
               <AppContainer />
               {<MyLoading
                     ref={(ref) => {
                        global.mLoadingComponentRef = ref;
                     }}
               />}
            </View>
         </Root>;
      }
   }

设置监听完成后，在支付页面，初始化商品列表。
::

   import { Card,Toast, CardItem, Body, Container, Button, Icon, Text, Content, CheckBox, Header, ListItem } from 'native-base';
   import RNIap, {
      purchaseErrorListener,
      purchaseUpdatedListener,
      type ProductPurchase,
      type PurchaseError
   } from 'react-native-iap';

   #productID列表
   const identifiers = [
      'com.xxx.1bill',
      'com.xxx.3bill',
   ];

   export default class Pay extends Component {
      constructor(props) {
         super(props);
         this.state = {
            pressed: 1,
            check_pay: 1,
            Username:"",
            Userid:0,
            Balance:0,
            Used:0,

            #使用prepared作为标志，在未完成getproducts或者purchase之前，不允许再次点击
            prepared:false,
            products:[], 
         };
      }

      #在componentwillmount或didmount中，调用getproducts
      async componentWillMount(){
         try {
            var products = await RNIap.getProducts(identifiers)

            #getproducts完成后 按钮可用
            this.setState({ products:products,prepared:true })
         } catch(err) {
               this.showtoast("Apple服务器无响应","warning")
         }
      }
      
      #按钮调用的支付函数中 直接使用requestPruchase（第二个参数为是否自动进行finishtransaction--false则为手动）
      async iospay(){
         try {
            const sku = "com.xxx.xxx"
            this.showtoast("正在向Apple发起支付请求,请耐心等待","success")
            await RNIap.requestPurchase(sku, false)
         } catch (err) {
            console.warn(err.code, err.message)
            this.showtoast("支付失败","warning")
         }
         this.setState({ prepared: true });
      }
      
      showtoast(text,type){
         Toast.show({
            text: text,
            buttonText: "好的",
            type: type
         });
      }
      render() {
         return (

            #.....
            #.....
            #.....

            <Grid style={{ marginTop: 10 }}>
               <Col>
                  <Button info block disabled={!this.state.prepared}
                        onPress={()=>{
                           this.setState({ prepared: false })
                           this.iospay()
                        }}
                  >
                     <Text>{this.state.prepared? "确认支付":"Connecting to AppStore.."}</Text>
                  </Button>
               </Col>
            </Grid>

         );
      }
   }


.. _header-n120:

服务端
~~~~~~

https://blog.csdn.net/endtheme_xin/article/details/83087223

这是Springboot 也就是 JAVA端实现IAP的一份代码（令人头秃）

https://blog.csdn.net/kaikai136412162/article/details/79670622

Python的一份，但是仅仅是测试，不是很全。

.. _header-n132:

这里我使用的是Springboot,首先抄了上面的IosVerifyUtil类,新增buyios请求

::

   @ResponseBody
   @PostMapping(value = "/buyios")
   @Transactional
   public String buyios(@RequestParam String username,@RequestParam String receipt) {
      
      String verifyResult = IosVerifyUtil.buyAppVerify(receipt, 1);            //1.先线上测试    发送平台验证
      if (verifyResult == null) {                                            // 苹果服务器没有返回验证结果
         System.out.println("无订单信息!");
      } else {                                                                // 苹果验证有返回结果
         JSONObject job = JSONObject.parseObject(verifyResult);               // Fastjson的JSONObject
         String states = job.getString("status");
         if ("21007".equals(states)) {                                            //是沙盒环境，应沙盒测试，否则执行下面
            verifyResult = IosVerifyUtil.buyAppVerify(receipt, 0);            //2.再沙盒测试  发送平台验证
            job = JSONObject.parseObject(verifyResult);
            states = job.getString("status");
         }
         //System.out.println(job);

         if (!"0".equals(states)) {
            return "fail";
         } else {// 前端所提供的收据是有效的    验证成功
            JSONObject returnJson = job.getJSONObject("receipt");
            JSONArray in_app = returnJson.getJSONArray("in_app");

            for (Object o : in_app) {
               JSONObject next = (JSONObject) o;

               //后台更新余额和订单信息
               
            }
            return "success";
         }
      }
      return "fail";
   }

fastjson的maven依赖
::

   <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.51</version>
   </dependency>

真机调试
--------

内购支付在模拟器上是永远失败的。

所以只能真机调试（谁买得起IOS呢，所以我借了一台）

如果是个穷人，可以选择 云真机（推荐Testin和Wetest，都有iphone真机）。
