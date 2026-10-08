1. 企业微信应用管理配置入口路径  `` 域名/pages/login/entryActive?channelCode=50007``
2. 在``/pages/login/entryActive``页面中请求

```
onLoad(options) {
		let channelCode = '';
		if (options.channelCode) {
			channelCode = options.channelCode;
		} else {
			channelCode = uni.getStorageSync('channelCode');
		}

		if (channelCode) {
			getChannelCode({ channelCode }).then((res) => {
				uni.setStorageSync('channelCode', channelCode);
				uni.setStorageSync('tenantId', res.data.tenantId);

				const redirectUrl = res.data.redirectUrl;

				const authRedirect = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${res.data.corpId}` + `&redirect_uri=${encodeURIComponent(redirectUrl)}&response_type=code&state=wxlogin` + `&scope=snsapi_privateinfo&agentid=${res.data.clientId}#wechat_redirect`;

				window.location.href = authRedirect;
			});
		} else {
			uni.showToast({
				title: '未获取到渠道信息,请退出重试',
				icon: 'none',
				duration: 3600,
			});
		}
	}
```

3. 在上方回掉页面中拿取``code``后进行登录请求换取token (*此页面路径必须与上方接口中回调页面redirectUrl一致)

```
<template>
	<view style="opacity: 1"> </view>
</template>

<script>
import { corpLogin } from '@/api/wx';

export default {
	data() {
		return {};
	},
	onLoad() {
		let code = '';
		const fullUrl = window.location.href; // 获取完整url
		// 获取code
		const queryString = fullUrl.split('?')[1];
		const parts = queryString.split('&');
		for (let part of parts) {
			if (part.startsWith('code=')) {
				code = part.substring(5);
				code = code.trim();
				break;
			}
		}

		let tenantId = uni.getStorageSync('tenantId');
		let channelCode = uni.getStorageSync('channelCode');

		if (code) {
			this.wxCorpLogin({
				code,
				state: 'wxlogin',
				grant_type: 'qywx',
				scope: 'all',
				source: 'WECHAT_ENTERPRISE',
				tenantId,
				channelCode,
			});
		}
	},
	methods: {
		wxCorpLogin(params) {
			corpLogin(params).then((data) => {
				this.$u.func.login(data, '/pages/plugin/workflow/pages/workbench/index');
			});
		},
	},
};
</script>

<style lang="scss" scoped>
</style>

```

