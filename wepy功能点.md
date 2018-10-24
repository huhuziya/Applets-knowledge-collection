
### 微信授权
```
<view class="component-submit-wrap">
    微信授权登录
    <button class="submit-btn" open-type="getUserInfo" bindgetuserinfo="authHandle"></button>
</view>

methods = {
	authHandle(res) {
		if (res.detail.userInfo) {
			wepy.navigateBack({
				delta: 1
			})
		} else {
			this.error()
		}
	}
}
```

### 轮播滚动
```
<view class="tips-list" style="{{'transform:' + transform}}">
transform() {
    return `translate3D(0, ${-100 * this.currentIndex}%, 0)`
}
```

### 触摸事件

```
bindtouchmove="touchMove"
bindtouchstart="touchStart"
bindtouchend="touchEnd"

 prev() {
    this.idx = this.idx > 0 ? this.idx - 1 : this.idx;
    this.$apply();
  }

  next() {
    if (Array.isArray(this.info.list) && this.info.list.length > 0){
        this.idx = this.idx <= this.info.list.length - 2 ? this.idx + 1 : this.idx;
        this.$apply();
    }
  }
  touchStart(event) {
    let touch = event.touches[0];
    this.startX = touch.pageX;
    this.$apply();
  }

  // 手指碰触slide并且滑动时执行
  touchMove(event) {
    let touch = event.touches[0];
    this.moveDist = touch.pageX - this.startX;
    this.$apply();
  }
  // 手指离开slide时执行
  touchEnd(event) {
    if (this.moveDist > 20) {
      this.prev();
    }

    if (this.moveDist < -20) {
      this.next();
    }
  }
```

### 长按保存图片

```
<image class="we-slide" 	
	wx:for="{{info.list}}"
	item="item" 
	index="index" 
	wx:key="key"
	src="{{item}}"
    bindlongtap="viewImage({{item}})"
></image>
methods={
    viewImage(_src) {
      wx.previewImage({
        current: _src, // 当前显示图片的http链接
        urls: [_src] // 需要预览的图片http链接列表
      });
    }
}


<image @tap="preview" class="content-wrap" src="{{qr}}"></image>
preview(){
    wx.previewImage({
        urls: [this.qr]
    })
},

```

### 保存邀请卡到手机
```
<view class="btn-right" hover-class="savetoPhone-active" @tap="save({{idx}})">保存邀请卡到手机</view>
//保存照片到手机
async save(_index) {
    this.$root.$parent.sensor({
        event_name: 'WxAppToolsButtonClick',
        param: {
            ButtonClick: 'SaveCard',
            Plan:Number(this.info.list[_index].plan_id),
            ActId:Number(this.info.list[_index].act_term_id),
            StyleId:Number(this.info.list[_index].style_id),
        },
    })

	wepy.showLoading()

	let res = await wepy.downloadFile({
		url: this.info.list[this.idx].data.invite_card
	}).catch((err) => {
		wepy.hideLoading()
	})

	if (!res) {
		return wepy.hideLoading()
	}

	let down = await wepy.saveImageToPhotosAlbum({
		filePath: res.tempFilePath
	}).then((res) => {
        wepy.hideLoading()
        wepy.showToast({
            title: '已保存到相册'
        })
    }).catch((err) => {
		wepy.hideLoading()
	})

	if (!down) {
		return wepy.hideLoading()
	}

	wepy.hideLoading()
},

```

### 一键复制
```
//复制功能
copy(_index) {
	wx.setClipboardData({
		data: this.info.list[_index].data.desc
	})
}
```

### 滑动图片改变文案
```
<p class="content-text"
   wx:for="{{info.list}}"
   item="item"
   index="index"
   wx:key="key"
   src="{{item.data.bg}}"
   wx:if="{{index === idx}}"
>
	{{item.data.desc}}
</p>
```

### 分享给好友
```
//实际开发中会发现这个 button 自带有样式，当背景颜色设置为白色的时候还有一个黑色的边框，刚开始那个边框怎么都去不掉，后来给button加了一个样式属性 plain="true" 以后，再在样式文件中控制样式 button[plain]{ border:0 } ，就可以比较随便的自定义样式了，比如说将分享按钮做成一个图标等
<button class="friend-btn" hover-class="shareFriend-active" open-type="share">
	送好友该课程
</button>

//分享给好友
onShareAppMessage() {
	return {
		title: this.info.list[this.idx].share.title, // 分享标题
		path: this.info.list[this.idx].share.path,
		imageUrl: this.info.list[this.idx].share.imageUrl
	}
}
```
[下文链接地址](https://blog.csdn.net/rolan1993/article/details/79754708)
```
//获取到转发目标群组信息，将用户和群组做某种绑定关系(openId + openGid)
1 onShareAppMessage: function () {
2 return {
3       title: '自定义转发标题',
4       path: '/page/user?id=123',
5       success: function(res) {
6 var shareTickets = res.shareTickets;
7 if (shareTickets.length == 0) {
8 return false;
9 }
10         wx.getShareInfo({
11             shareTicket: shareTickets[0],
12             success: function(res){
13 var encryptedData = res.encryptedData;
14 var iv = res.iv;
15 }
16 })
17 },
18       fail: function(res) {
19 // 转发失败
20 }
21 }
22 }
```

### 微信自带弹窗

```
wx.showModal({
    title: "领取失败"
});

wepy.showToast({
    title: '发送成功',
    icon: 'success',
    duration: 2000
})
```



### 小程序间跳转
```
<navigator class="btn" target="miniProgram" app-id="{{info.app_id}}" path="{{info.path}}">点击跳转</navigator>
```



### 循环定时器
```
async onShow() {
    await this.sendGetStationReq();
    this.isShow = this.info.show_toast;
    this.$apply()
    console.log(new Date().getTime())
    console.log(this.renderList[this.idx].qualification.active_timestamp)

    if (new Date().getTime() - this.renderList[0].qualification.active_timestamp > 0) {
        this.isQualification[0] = true;
        this.$apply();
    }

    this.timer = setInterval(()=> {
      if (new Date().getTime() - this.renderList[this.idx].qualification.active_timestamp > 0) {
        this.isQualification[this.idx] = true;
        this.$apply();
      }
    }, 100);

}

onHide() {
    clearInterval(this.timer)
    this.timer = null
}

onUnload(){
    clearInterval(this.timer)
    this.timer = null
}
```

### 页面滚动触发事件的处理函数
```
onPageScroll() {
    wx.createSelectorQuery().selectAll('.components-giving-wrap').boundingClientRect((rects) => {
        if (rects[0].top + rects[0].height <= 0) {
            this.showBtn = true
        } else {
            this.showBtn = false
        }
        this.$apply()
    }).exec()
}
```
