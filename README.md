# 组件通信的几种方法

## 最佳实践

* 当处理较复杂，多级组件嵌套传递数据时，使用 vuex 与 localStorage
* 当单纯子孙等组件之间的传递，使用 $attrs 与 $listeners
* 当跨级或平级稍多的情况下，可以用eventbus
* 当父子组件时，props与$emit、$ref、$children、$parent、$root

## props和$emit

* props

        /**父->子
        *  父组件绑定数据在模板的子组件中
        *  子组件通过props属性获取
        *
        */
        Vue.component("Child", {
            template: `<p> {{msg}} </p>`,
            props: ["msg"]
        })

        new Vue({
            data() {
                return {
                    msg: "我是爸爸传给儿子的信息"
                }
            },
            template: `<Child :msg="msg"/>`,
        }).$mount("#app")

* $emit

        /**
        *  子->父
        *  子组件中利用事件发送$emit
        *  父组件的方法名称与$emit的方法名称相同，且无需添加变量 @send-msg='getMsg'
        */
        Vue.component("Child",{
            data(){
                return{
                    msg:""
                }
            },
            template:`
                <input @keydown.enter="sendMsg(msg)" v-model='msg'>
            `,
            methods: {
                sendMsg(msg){
                    // console.log(this)
                    this.$emit('send-msg',msg)
                }
            }
        })

        let Parent={
            data(){
                return{
                    // msg:""
                }
            },
            template:`
                <Child @send-msg='getMsg'/>
            `,
            methods:{
                getMsg(msg){
                    console.log(msg)
                }
            }
        }

## Eventbus(中央事件总线)

* 具体实现方式：

        /**
        *   创建一个空的Vue实例
        *   在传递数据的组件方法中emit
        *   在接受数据的组件方法中on,接收数据在created或mounted中获取
        */
        var Event=new Vue();
        Event.$emit(事件名,数据);
        Event.$on(事件名,data => {});

## $attrs与$listeners

    /**
    * $attrs:包含了父作用域中不被 prop 所识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件。通常配合 inheritAttrs 选项一起使用。
    *  之后通过子组件的created或mounted获取
    *
    * $listeners：包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件
    *   与子传父相同，emit发送事件信息，接收信息的高级组件 @ 获取
    *   每一代传递都用v-on='$listeners'
    */

## provide与inject

    /**
    *   父组件provide提供，
    *   子组件inject注入
    */

    // A.vue
    export default {
      provide: {
        name: '浪里行舟'
      }
    }
    // B.vue
    export default {
     inject: ['name'],
     mounted () {
       console.log(this.name);  // 浪里行舟
     }
    }

## $parent、$root与$children、$refs实例

* $parent：最近一个父节点
* $root:查找根组件，并可以配合$children遍历全部组件 `this.$root.$children[0]`
* $children:当前组件的直接子组件，可以遍历全部子组件， 需要注意 $children 并不保证顺序，也不是响应式的
* $refs:对属性或者组件 `ref="a"` ，若对组件则 `this.$refs.a`为组件实例

