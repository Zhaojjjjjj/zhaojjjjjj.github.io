> 开发者需要登录[uniCloud控制台](https://unicloud.dcloud.net.cn/pages/uni-login/login-account)，申请开通一键登录服务。
> 详细步骤参考：[一键登录服务开通指南](https://doc.dcloud.net.cn/uniCloud/uni-login/service)
- 开通uni一键登录服务后，需要等审核通过后才能正式使用。在审核期间可以使用HBuilder标准基座真机运行调用一键登录功能，调用时会从你的账户中扣费；但在审核期间不可以使用自定义基座调用一键登录功能，调用时会返回错误。

### 1. 首先开通uniCloud服务空间
- 开通[uniCloud服务空间](https://unicloud.dcloud.net.cn/pages/login/login)
- 一键登录需要uniCloud，但并不要求开发者把所有的后台服务都迁移到uniCloud

### 2. 对项目点右键，创建uniCloud开发环境，然后绑定到上一步创建的服务空间上

### 3. 对uniCloud/cloudfunctions/点右键，创建云函数

- 例如云函数名称设置为login
> login/index.js代码
> 自HBuilderX 3.4.0起云函数需启用uni-cloud-verify扩展之后才可以调用getPhoneNumber接口
```js
'use strict';
exports.main = async (event, context) => {
    try {
        const res = await uniCloud.getPhoneNumber({
			appid: '', //appid名称
            provider: 'univerify',
            access_token: event.access_token,
            openid: event.openid
        });

        return {
            code: 0,
            data: res,
            message: '获取手机号成功'
        };
    } catch (error) {
        return {
            code: -1,
            message: '获取手机号失败',
            error: error.message || '未知错误'
        };
    }
}

```

> login/package.json代码
```js
{
  "name": "login",
  "extensions": {
    "uni-cloud-verify": {} // 启用一键登录扩展，值为空对象即可
  }
}
```

### 4. 前端登录页面代码
```js
uni.login({
	provider: 'univerify',
	univerifyStyle: {
		fullScreen: true,
	},
	success(res) {
		uniCloud.callFunction({
			name: 'login',
			data: {
				access_token: res.authResult.access_token,
				openid: res.authResult.openid,
			}
		}).then((dataRes) => {
			console.log("获取到的手机号数据", dataRes.result.data.phoneNumber);
		        })
	        })
	}
});
```

### 5. 对云函数点右键，上传到服务空间，每次修改云函数后都需要上传到服务空间

### 6. Manifest.json — 安卓/ios模块设置 — 勾选OAuth(登录鉴权)和一键登录(univerify)

### 7. [云服务空间](https://unicloud.dcloud.net.cn/pages/login/login) — 安全 — 安全网络
- 新增刚添加的应用

### 8. uni一键登录的授权弹出界面是默认是半屏的，也可以配置为全屏。这个界面本质是运营商sdk弹出的，授权弹出界面可以通过 univerifyStyle 设置有限定制
```css
{
    "fullScreen": false, // 是否全屏显示，默认值： false
    "backgroundColor": "#ffffff",  // 授权页面背景颜色，默认值：#ffffff
    "backgroundImage": "", // 全屏显示的背景图片，默认值："" （仅支持本地图片，只有全屏显示时支持）
    "icon": {
        "path": "static/xxx.png", // 自定义显示在授权框中的logo，仅支持本地图片 默认显示App logo
        "width":  "60px",  //图标宽度 默认值：60px
        "height": "60px"   //图标高度 默认值：60px
    },
    "closeIcon": {
        "path": "static/xxx.png", // 自定义显示在授权框中的logo，仅支持本地图片
        "width":  "60px",  //图标宽度 默认值：60px (HBuilderX 4.0+ 仅iOS支持)
        "height": "60px"   //图标高度 默认值：60px (HBuilderX 4.0+ 仅iOS支持)
    },
    "phoneNum": {
        "color": "#202020"  // 手机号文字颜色 默认值：#202020
    },
    "slogan": {
        "color": "#BBBBBB"  //  slogan 字体颜色 默认值：#BBBBBB
    },
    "authButton": {
        "normalColor": "#3479f5", // 授权按钮正常状态背景颜色 默认值：#3479f5
        "highlightColor": "#2861c5",  // 授权按钮按下状态背景颜色 默认值：#2861c5（仅ios支持）
        "disabledColor": "#73aaf5",  // 授权按钮不可点击时背景颜色 默认值：#73aaf5（仅ios支持）
        "textColor": "#ffffff",  // 授权按钮文字颜色 默认值：#ffffff
        "title": "本机号码一键登录", // 授权按钮文案 默认值：“本机号码一键登录”
        "borderRadius": "24px"	// 授权按钮圆角 默认值："24px" （按钮高度的一半）
    },
    "otherLoginButton": {
        "visible": true, // 是否显示其他登录按钮，默认值：true
        "normalColor": "", // 其他登录按钮正常状态背景颜色 默认值：透明
        "highlightColor": "", // 其他登录按钮按下状态背景颜色 默认值：透明
        "textColor": "#656565", // 其他登录按钮文字颜色 默认值：#656565
        "title": "其他登录方式", // 其他登录方式按钮文字 默认值：“其他登录方式”
        "borderColor": "",  //边框颜色 默认值：透明（仅iOS支持）
        "borderRadius": "0px" // 其他登录按钮圆角 默认值："24px" （按钮高度的一半）
    },
    "privacyTerms": {
        "defaultCheckBoxState":true, // 条款勾选框初始状态 默认值： true
        "isCenterHint":false, //未勾选服务条款时点击登录按钮的提示是否居中显示 默认值: false (3.7.13+ 版本支持)
        "uncheckedImage":"", // 可选 条款勾选框未选中状态图片（仅支持本地图片 建议尺寸 24x24px）(3.2.0+ 版本支持)
        "checkedImage":"", // 可选 条款勾选框选中状态图片（仅支持本地图片 建议尺寸24x24px）(3.2.0+ 版本支持)
        "checkBoxSize":12, // 可选 条款勾选框大小
        "textColor": "#BBBBBB", // 文字颜色 默认值：#BBBBBB
        "termsColor": "#5496E3", //  协议文字颜色 默认值： #5496E3
        "prefix": "我已阅读并同意", // 条款前的文案 默认值：“我已阅读并同意”
        "suffix": "并使用本机号码登录", // 条款后的文案 默认值：“并使用本机号码登录”
        "privacyItems": [  // 自定义协议条款，最大支持2个，需要同时设置url和title. 否则不生效
            {
                "url": "https://", // 点击跳转的协议详情页面
                "title": "用户服务协议" // 协议名称
            }
        ]
    },
    "buttons": {  // 自定义页面下方按钮仅全屏模式生效（3.1.14+ 版本支持）
        "iconWidth": "45px", // 图标宽度（高度等比例缩放） 默认值：45px
        "list": [
            {
                "provider": "apple",
                "iconPath": "/static/apple.png" // 图标路径仅支持本地图片
            },
            {
                "provider": "weixin",
                "iconPath": "/static/wechat.png" // 图标路径仅支持本地图片
            }
        ]
    }
}
```