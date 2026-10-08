```
<template>
	<u--form labelPosition="left" labelWidth="100" :model="form" :rules="rules" ref="uForm" errorType="border-bottom">
		<u-form-item label="附件" prop="enclosureList" borderBottom>
			<u-upload :fileList="fileList1" @afterRead="afterRead" @delete="deletePic" name="1" multiple :maxCount="10" :disabled="disabled" :deletable="!disabled"></u-upload>
		</u-form-item>		
	</u--form>

	<view style="width: 0px; height: 0px; overflow: hidden">
		<canvas canvas-id="cid" :style="{ height: `${cvHeight}px`, width: `${cvWidth}px` }"></canvas>
        </view>
</template>

<script>
import { options } from '@/http/config.js';
import { clientId, clientSecret } from '@/common/setting';
import { Base64 } from '@/utils/base64.js';
import { bfAdd, glsjSelect, nbpfrSelect, bfDetail } from '@/api/business.js';

export default {
	data() {
		return {
			form: {
				enclosureList: [],
			},
			beforeAction: options.baseURL + '/api/web-service/oss/endpoint/put-file',
			token: {
				'Blade-Auth': uni.getStorageSync('accessToken'),
				Authorization: 'Basic ' + Base64.encode(clientId + ':' + clientSecret),
			},
			disabled: false,
			fileList1: [],

			cvHeight: 0,
			cvWidth: 1024,
			waterMarkerAddress: '',
		};
	},

		// upload
		async afterRead(event) {
			let lists = [].concat(event.file);
			let Images = [];

			for (let i = 0; i < lists.length; i++) {
				const item = lists[i];
				Images.push(item);
			}

			let fileListLen = this.fileList1.length;

			// 将状态设置为上传中
			Images.forEach((item) => {
				this.fileList1.push({
					...item,
					status: 'uploading',
					message: '上传中',
				});
			});

			for (const image of Images) {
				const tpurl = image.url; // 使用Images数组

				const imageInfo = await this.getImageInfo(tpurl);
				this.cvWidth = 1024;
				this.cvHeight = (this.cvWidth * imageInfo.height) / imageInfo.width;
				// 使用当前图像的信息来绘制
				await this.imgToCanvas(tpurl, this.cvWidth, this.cvHeight, fileListLen);
				fileListLen++; // 更新文件列表索引
			}
		},

		getImageInfo(url) {
			return new Promise((resolve, reject) => {
				uni.getImageInfo({
					src: url,
					success: resolve,
					fail: reject,
				});
			});
		},

		async imgToCanvas(url, width, height, fileListLen) {
			const nowDate = new Date();
			const year = nowDate.getFullYear();
			const month = String(nowDate.getMonth() + 1).padStart(2, '0');
			const day = String(nowDate.getDate()).padStart(2, '0');
			const week = String(nowDate.getDay());
			const hour = String(nowDate.getHours()).padStart(2, '0');
			const minute = String(nowDate.getMinutes()).padStart(2, '0');

			return new Promise(async (resolve) => {
				const ctx = uni.createCanvasContext('cid');

				// // 清空画布
				// ctx.clearRect(0, 0, width, height);

				// 绘制原始图像
				ctx.drawImage(url, 0, 0, width, height);

				// // 绘制水印图片
				// const watermarkImage = await this.loadImage('./../../static/images/logo.jpg');
				// const watermarkWidth = 150;
				// const watermarkHeight = 150;
				// const watermarkX = width - 170; // 水印图片在 Canvas 上的 x 坐标
				// const watermarkY = height - 200; // 水印图片在 Canvas 上的 y 坐标
				// ctx.drawImage(watermarkImage, watermarkX, watermarkY, watermarkWidth, watermarkHeight);

				// 绘制文字水印
				ctx.fillStyle = '#ddd';
				ctx.fillRect(20, height - 210, 6, 200);
				ctx.font = '40px Arial';
				ctx.fillText(`${hour}:${minute}`, 35, height - 90); // 时分
				ctx.fillText(`${year}.${month}.${day}`, 35, height - 130);
				ctx.fillText(uni.getStorageSync('lifeData').userInfo.real_name, 35, height - 170);

				ctx.fillText(this.waterMarkerAddress, 35, height - 20);

				ctx.draw();

				setTimeout(() => {
					uni.canvasToTempFilePath(
						{
							canvasId: 'cid',
							width: width,
							height: height,
							destWidth: width,
							destHeight: height,
							success: async (res) => {
								resolve(res.tempFilePath);
								await this.setFilePath(res.tempFilePath, fileListLen);
								ctx.clearRect(0, 0, width, height);
								ctx.draw();
							},
						},
						this,
					);
				}, 100);
			});
		},
		async loadImage(url) {
			return new Promise((resolve, reject) => {
				uni.getImageInfo({
					src: url,
					success: (res) => {
						resolve(res.path);
					},
					fail: (err) => {
						reject(err);
					},
				});
			});
		},
		uploadFilePromise(url) {
			return new Promise((resolve, reject) => {
				uni.uploadFile({
					url: `${options.baseURL}/api/web-service/oss/endpoint/put-file`,
					filePath: url,
					name: 'file',
					header: {
						'Blade-Auth': uni.getStorageSync('accessToken'),
						Authorization: 'Basic ' + Base64.encode(clientId + ':' + clientSecret),
					},
					success: (res) => {
						let data = JSON.parse(res.data);
						resolve(data.data.link);
					},
				});
			});
		},
		async setFilePath(path, fileListLen) {
			const result = await this.uploadFilePromise(path);

			let item = this.fileList1[fileListLen];
			this.fileList1.splice(
				fileListLen,
				1,
				Object.assign(item, {
					status: 'success',
					message: !result ? '上传失败' : '',
					url: result,
				}),
			);
		},

		// del img
		deletePic(event) {
			this.fileList1.splice(event.index, 1);
		},

		getWaterMarkerAddress() {
			const key = '4198af3b4995df4cd919d17dcfcfb410';

			uni.request({
				url: 'https://restapi.amap.com/v3/ip?key=' + key, // IP定位接口
				method: 'GET',
				success: (res) => {
					if (res.data.status === '1') {
						const city = res.data.city; // 获取城市信息
						const province = res.data.province; // 获取省份信息
						this.waterMarkerAddress = `${province}${city}`; // 地址信息
					}
				},
				fail: (error) => {
					console.error('请求失败：', error);
				},
			});
		},
	},
};
</script>

```

