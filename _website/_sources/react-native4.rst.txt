.. post::Sep 10 , 2019
   :tags: directive
   :category: react-native
   :author: HicoderDR


react-native初识:props和state
#########################################
Component内部结构
*******************************
回顾标准Component：
+++++++++++++++++++
::

    import React, { Component } from 'react';
    import { Text } from 'react-native';
    //以上为引入的组件

    export default class HelloWorldApp extends Component{
        //定义构造函数 和 成员变量
        constructor(props){
            super(props);
            this.state={

            };
        }     
        //定义其他函数
        Test(){
            //你想写的内容
        }  
        //渲染函数
        render(){
            //你想写的内容
            //返回值
            return(
                //组件真正渲染的视图
                <Text>Hello World</Text>
            );
        }
    }

props(属性)
++++++++++++++
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
            //你的内容
        }
        fuc2=()=>{
            //你的内容
        }
        render(){
            var x="myname";                 //定义一个x
            return(
                <Son
                    name={this.x}           //传递值给Son
                    fuc1={ (parm)=>{
                        this.fuc1(parm); 
                    }}                      //传递函数方法 第一种写法“传调用”
                    fuc2={this.fuc2}        //传递函数方法 第二种写法“传引用”
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
            super(props);                     //这时就继承了props
        }
        render(){
            return(
                <Text>{this.props.name}</Text>
            );
        }
    }

state(状态)
++++++++++++++