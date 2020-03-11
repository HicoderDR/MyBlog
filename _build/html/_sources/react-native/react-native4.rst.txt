.. post::Sep 20 , 2019
   :tags: react-native
   :category: react-native
   :author: HicoderDR

react-native初识: `state <https://reactnative.cn/docs/state/>`_ 和 `props <https://reactnative.cn/docs/props/>`_
############################################################################################################################
Component内部结构
*******************************
回顾标准Component：
+++++++++++++++++++
::

    import React, { Component } from 'react';
    import { Text } from 'react-native';
    #以上为引入的组件

    export default class HelloWorldApp extends Component{
        #定义构造函数 和 成员变量
        constructor(props){
            super(props);
            this.state={

            };
        }     
        #定义其他函数
        Test(){
            #你想写的内容
        }  
        #渲染函数
        render(){
            #你想写的内容
            #返回值
            return(
                #组件真正渲染的视图
                <Text>Hello World</Text>
            );
        }
    }

`state(状态) <https://reactnative.cn/docs/state/>`_ 
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
首先，state的用处是什么呢？主要和 
`Component生命周期 <https://www.jianshu.com/p/c21e0314beef>`_
有关。

当某些值变化的时候，我们希望能重新render（刷新界面）
，这就是state最基本的用途。

.. image:: ../_static/life.png 

这些生命周期函数，我们都是可以覆写的，比如说
::

    class A extends Component{
        componentDidMount(){
            getData(); 
            #初次进入界面的时候需要做的事情
            #比如请求数据
        }
        render(){
            #......
        }
    }

State是我们刷新界面的便捷接口，
我们只要通过setState()方法，
更改state的同时就自动重新渲染
::

    class A extends Component{
        constructor(props){
            super(props);
            this.state={
                text:"init",
                data:[],
                #定义state
            }
        }
        render(){
            <View>
                <Text>
                    {this.state.text}
                </Text>
                <Button 
                    title="点我！"
                    onPress={()=>{
                        this.setState({
                            text:"mod"
                        });
                    }}
                />
            </View>
        }
    }



`props(属性) <https://reactnative.cn/docs/props/>`_
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
就如我们熟知的面向对象思想，这就是所谓的类成员，
它可以是一个变量，也可以是一个函数。

父组件向子组件传递props
--------------------------
父组件中
::
    
    import React, { Component } from 'react';
    import { Text } from 'react-native';
    import Son from './Son'

    export default class Father extends Component{
        fuc1(parm){
            #你的内容
        }
        fuc2=()=>{
            #你的内容
        }
        render(){
            var x="myname";                 #定义一个x
            return(
                <Son
                    name={this.x}           #传递值给Son
                    fuc1={ (parm)=>{
                        this.fuc1(parm); 
                    }}                      #传递函数方法 第一种写法“传调用”
                    fuc2={this.fuc2}        #传递函数方法 第二种写法“传引用”
                />
            );
        }
    }

子组件中

::

    import React, { Component } from 'react';
    import { Text } from 'react-native';

    export default class Son extends Component{
        constructor(props){
            super(props);                     #这时就继承了props
        }
        render(){
            return(
                <Text>{this.props.name}</Text>
            );
        }
    }

子组件向父组件传递props
--------------------------
父组件中
::
    
    import React, { Component } from 'react';
    import { Text } from 'react-native';
    import Son from './Son'

    export default class Father extends Component{
        constructor(props){
            
        }
        _set(year) {
            this.setState({
                year:year,
            });
        }
        render(){
            var x="myname";                 #定义一个x
            return(
                <Son
                    _set={this._set.bind(this)}
                    #将父组件函数与子组件绑定
                />
                <Text>
                    {this.state.year}
                </Text>
            );
        }
    }

子组件中

::

    export default class Son extends Component {
        render() {
            return (
                <View>
                    <Text
                        onPress={() => {
                            this.props._set(year);
                        }}  
                    >
                    点击
                    </Text>
                </View>
            );
        }
    }
