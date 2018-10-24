### 声明
```
引入：import wepy from 'wepy'

//声明一个app小程序实例
export default class MyApp extend wepy.app{}

//声明一个page页面实例
export default class IndexPage extend wepy.page{}

在page页面可通过this.$parent来访问app实例

methods属性只能声明页面wxml标签的bind、catch事件，不能声明自定义方法
普通自定义方法于methods同级

//声明一个component组件实例
export default class MyComponent extend wepy.component{}
组件的所有业务功能都在组件内开发
```

### prop 向子组件传递数据
1.数据双向绑定 twoway:true

2.父组件一定要有的格式：syncTitle.sync = "parentTitle"
```
//主页面
<view :info.sync="item" ></view>
//子组件
props = [
    status:{
        type:Number,
        twoway:true，
        default:0
    }
]
```
3.数据传值prop只能传两层，第三层要加一个数组包含起来
```
<view class="detail">
		<repeat for="{{[info]}}"
            item="item"
            index="index"
            key="key">
            <view class="detail-item">
                <SliderShow :res.sync="item"></SliderShow>
            </view>
        </repeat>
</view>

//不要直接使用以下写法
<view class="detail">
    <SliderShow :res.sync="item"></SliderShow>
</view>
```

### 组件的引用
```
<template>
    <repeat for="{{ list }}" key="index" index="index" item="item">
        <child :item="item"></child>
    </repeat>
</template>
<script>
import wepy from 'wepy';
import Child from '../components/child';

    export default class Index extends wepy.component {
        //声明组件，分配组件id为child
        components = {
            child: Child
        };
    }
</script>
```

### 组件间的通信

子组件：
```
<view @tap="showModal({{info}})"></view>
methods={
    showModal(info){
        this.$emit("controlModal",info.isShowModal)
    }
}
```
父组件：
```
events={
    controlModal(isShow){
        this.isShow = isShow
    }
}
```
```
$broadcast:由父组件发起的，经广度优先顺序搜索
$emit：从子组件依次发送给其父组件
$invoke:是一个页面或组件对另一个组件中的方法的直接调用
this.$invoke('ComA', 'someMethod', 'someArgs');
```

### 组件自定义事件处理函数
```
页面：

<child @childFn.user="parentFn"></child>
 methods = {
    parentFn (num, evt) {
    console.log('parent received emit event, number is: ' + num)
    }
}

子组件：

<view @tap="tap">Click me</view>
tap () {
    console.log('child is clicked')
    this.$emit('childFn', 100)
}
```

### 事件绑定
```
<text wx:if="{{ isTextShow }}">{{  text  }}</text>
<button type="primary" btntap="btnClick">{{ btnText }}</button>
page({
    data:{
        text:"初始内容",
        btnText:"按钮内容",
        isTextShow:"true",
    },
    btnClick:function(){
        this.setData({btnText:"按钮被点击过了"})
        var isShow = this.data.isTextShow
        this.setData({text:"点击按钮，text内容修改了",isTextShow:!isShow})
    }
})
```
//子组件：
```	
<li
    class="tab-item"
    wx:for="{{tabs}}" wx:key="index" index="index" item="item"
    :class="{ active: currentTab == item.value }"
    @tap="clickTab({{item.value}})"
>
   {{ item.title }}
</li>
methods = {
	clickTab(_value) {
	this.currentTab = _value;
	console.log(_value)
	this.$emit('tabClick', _value);
   },
}
```
//父组件：
```
events = {
    tabClick(val){
        console.log(1)
        this.value = val
        this.$apply()
    }
}
//无需再再标签里面写@tabClick="handleClick"
```




### 页面跳转
1.页面间的跳转
```
//跳转后可返回（先调用index生命周期方法onhide，再调用logs页面的onload--onshow--onready）
itemClick:function(){
    wx.navigateTo({
        url:'../logs/logs'
    })
}
//跳转后不可返回,index页面被销毁（先调用index生命周期方法onhide--onupload，再调用logs页面的onload--onshow--onready）
itemClick:function(){
    wx.redirectTo({
        url:'../logs/logs'
    })
}
//页面间的跳转
this.$root.$navigate('../../page/activity/index', {
    plan_id: this.$wxpage.options.plan_id,
    award_id: info.award.id
 })
```

2.页面间的参数传递

```
wx:navigate({
    url:"../logs/logs?id=1&title=标题abc"
})
//logs页面中onload中通过options传入的参数id和title
onLoad:function(options){
    this.articleId:options = id
    this.pageTitle:options = title
}

```
3.页面初始化 options为页面跳转所带来的参数
```
onLoad(options) {
    this.value = options.value ? options.value : 'give'
    this.$apply()
}
```


### 发送请求

```
    //原生写法发送请求
    async sendPostDrawReq() {
        await this.$root.$parent.req
        .send({
            url: this.$root.$parent.req.api.collect,
            method: "post",
            data: {
            toolbox_id: this.toolboxid,
            plan_id: this.planid,
            award_id: this.awardid
            }
        })
        .then(res => {
            this.errorStatus = res.errcode;
        })
        .catch(e => {
            wx.showModal({
            title: "领取失败",
            content: e
            });
        });
    }
    
    //get请求
    async sendGetAwardReq() {
        let { data } = await this.$root.$parent.req.send({
            url: this.$root.$parent.req.api.award,
            method: "get",
            data: this.param
        });
    
        this.award = data;
        this.$apply();
    }
    async sendGetTestReq() {
        let {data} = await this.$root.$parent.req.send({
            url: this.$root.$parent.req.api.test,
            data: Object.assign(this.$wxpage.options, {
                scene_id: this.$wxpage.options.scene_id	
                //query参数，接收从别的页面传递过来的query参数
            }),
            restful: {
                act_term_id: this.$wxpage.options.act_term_id	
                //必须参数。url的/{act_term_id}/index
            }
        })

        this.questions = data.questions
        this.next = data.next
        this.$apply()
    }

    let {data} = await this.$root.$parent.req.send({
        url: this.$root.$parent.req.api.activity,
        method: 'get',
        data: Object.assign(this.$wxpage.options, {
            award_id: this.$wxpage.options.award_id	
            //query参数，接收从别的页面传递过来的query参数
        }),
    })
```



### 表单知识点
1.实时监听input输入组件
```
bindinput="inputPhoneNum"
inputPhoneNum(e) {
    this.phoneNum = e.detail.value
    this.$apply()
}

```
2.input属性
```
confirm-type属性定义右下角按钮的文本（在Android上测试无效）
auto-focus:true
    类型：Boolean
    自动聚焦，拉起键盘。如果你设置为true,那边你打开页面就会弹出键盘，这时可能会遮挡一些控件。
    此功能即将废弃，尽量使用focus
focus:
    类型：boolean
    获取焦点
```

### for循环
```
<view wx:for="{{ news }}" wx:for-item="newsItem">
    {{ index }}:{{ newsItem }}
</view>
```

### 点击按钮触发样式改变
//hover-class
```
 <view 
    wx:if="{{send||phoneStatus}}" 
    class="sendMsg" 
    hover-class="active" 
    @tap="sendMsg()"
>获取验证码</view>   
.active{
    background-color: #d43d4d;
}
```
