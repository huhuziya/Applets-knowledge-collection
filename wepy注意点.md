
### json数组处理
```
//在页面onShow的时候获取数组中同一属性的集合

    data.award_records.forEach(item =>{
        this.awardtimes.push(moment(item.timestamp, 'YYYY年MM月DD日'))
    });
    
   data.invitees.filter(item =>{
       this.invitetimes.push(moment(item.timestamp, 'YYYY年MM月DD日'))
    });

```

### 判断数据是否为空
```
if (!Array.isArray(val) || !val.length){
    //...
}
if(info!=null){
    //...
}
```

### 行内样式书写规范
```
style="transform: translateX({{idx * 100}}%)"（坑 ：不要用this.dix）
style="background-color:{{item.bg_color}}"
```

### 轮播图点击事件出现同步控制 
```
//讲点击改变的数据传到父组件，改变最初请求的数据达到控制目的
//repeat一个子组件，child子组件有一个tap事件（让一段文字变色），点击事件所有子组件都会变色

组件是在编译阶段编译进页面的，每个组件都是唯一的一个实例，目前只提供简单的 repeat 支持。
不支持在 repeat 的组件中去使用 props, computed, watch 等等特性
```
```
<!-- 错误使用 -->
// list.wpy
<view>{{test.name}}</view>

// index.wpy
<repeat for="{{mylist}}">
   <List :test.sync="item"></List>
</repeat>

<!-- 推荐用法 -->
// list.wpy
<repeat for="{{mylist}}">
    <view>{{item.name}}</view>
</repeat>

// index.wpy
<List :mylist.sync="mylist"></List>
```

//例子：

//slidershow子组件
```
<view class="left-wrap btn-wrap" @tap="left({{res}})" wx:if="{{res.idx>0}}">
    <view class="btn-left buttons">
        <image src="" class="left-pic pic"/>
    </view>
</view>
```

//redemm组件
```
<SliderShow :res.sync="item"
    :numbers.sync="numbers"
    @changeIdx.user="changeIdx"
    @changeNum.user="changeNum"
></SliderShow>
changeIdx(res, method) {
    this.$emit("changeIdx", res, method)
}
```

//home页
```
<view class="component-redemm-wrap">         <Redemm:info.sync="item":total.sync="info"@changeIdx.user="changeIdx" ></Redemm>
</view>
changeIdx(res, method) {
  if (method === "plus") {
    this.info.activities[res.index].idx =
      this.info.activities[res.index].idx + 1;
  } else {
    this.info.activities[res.index].idx =
      this.info.activities[res.index].idx - 1;
  }

  this.$apply();
},
```

### wepy的组件里只有一个view标签的视图、不能包含其他组件！！

```
//错误写法：
<template>
    <view>
    ....
    </view>
    <ApplyEntrance></ApplyEntrance>
</template>

//正确写法：
<template>
    <view>
        ....
        <ApplyEntrance></ApplyEntrance>
    </view>
</template>
```
