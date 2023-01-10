<template>
	<view>
		<!-- <u-navbar :is-back="false" title="　" :border-bottom="false">
			<view class="u-flex u-row-right" style="width: 100%;">
				<view class="camera u-flex u-row-center">
					<u-icon name="camera-fill" color="#000000" size="48"></u-icon>
				</view>
			</view>
		</u-navbar> -->
		<view class="u-flex user-box u-p-l-30 u-p-r-20 u-p-b-30">
			<view class="u-m-r-10">
				<u-avatar :src="avatarUrl" size="100"></u-avatar>
			</view>
			<view class="u-flex-1">
				<!-- <view class="u-font-18 u-p-b-20">uView ui</view>
				<view class="u-font-14 u-tips-color">微信号:helang_uView</view> -->
			</view>
			<view class="u-m-l-10 u-p-10" on>
				<button type="default" @chooseavatar="onChooseAvatar" open-type="chooseAvatar">

					<u-icon name="camera-fill" color="#969799" size="58"></u-icon>
				</button>

			</view>
			<!-- <view class="u-m-l-10 u-p-10">
				<u-icon name="arrow-right" color="#969799" size="28"></u-icon>
			</view> -->
			<!-- <button type="default" open-type="getPhoneNumber" @getphonenumber="decryptPhoneNumber">获取手机号</button> -->
		</view>

		<view class="u-m-t-20">
			<u-cell-group>
				<u-cell-item icon="server-man" title="昵称:">
					<u-input @blur="onNickName" placeholder="请选择昵称" v-model="nicKname1" type="nickname"></u-input>
				</u-cell-item>

			</u-cell-group>
		</view>
		<view class="u-m-l-10 u-p-10" on>
			<button type="default" @click="save">
				保存
			</button>

		</view>
	</view>

</template>
<script>
	var _this;
	import {
		request
	} from '@/common/js/request'
	export default {
		data() {
			return {
				avatarUrl: "",
				nicKname1: "",
				openId: "",
				userInfo:""
			}
		},
		onLoad(options) {
			_this = this;
			_this.openId = options.openid;
			_this.userInfo = uni.getStorageSync('userInfo')
			console.log(111111,_this.userInfo.avatar)
			
		},
		methods: {
			onNickName(event) {
				console.log(event)
				_this.nicKname1 = event;
			},
			onChooseAvatar(e) {
				console.log(e)
				_this.avatarUrl = e.detail.avatarUrl;

			},
			async save() {
				if (!_this.avatarUrl) {
					uni.showToast({
						icon: "error",
						title: "请选择微信头像",
						duration: 2000
					});
					return
				}
				// console.log('_this.nickname', _this.nicKname1)
				if (!_this.nicKname1) {
					uni.showToast({
						icon: "error",
						title: "请选择微信昵称",
						duration: 2000
					});
					return
				}
				const result = uniCloud.uploadFile({
					filePath: _this.avatarUrl,
					// cloudPath: String(Math.random() * 5).split('.')[1] + '.png',
					cloudPath: _this.openId + '.png'
				})
				_this.userInfo = uni.getStorageSync('userInfo')
				// console.log(22222,_this.userInfo.avatar)
				// uniCloud
				// 	.deleteFile({
				// 		fileList: [_this.userInfo.avatar]
				// 	})
				// 	.then(res => {
				// 		// console.log(888,res)
				// 	});
				result.then(res => {
					result.then(res => {
						uniCloud.callFunction({
							name: 'tihuo',
							data: {
								operation: 'upweixininfo',
								data: {
									openId: _this.openId,
									avatarUrl: res['fileID'],
									nickName: _this.nicKname1,
								}
							}
						}).then(res => {
							if (res.result.code == 200) {
								uni.setStorageSync(
									'userInfo',
									res.result.data);
								uni.showToast({
									icon: "success",
									title: "上传成功",
									duration: 2000
								});
								uni.switchTab({
									url: '/pages/index/index'
								});
							}
						}).catch((err) => {
							uni.showToast({
								icon: "error",
								title: "保存失败",
								duration: 2000
							});
						})

					})



					// const res = await request('tihuo', 'upweixininfo', {
					// 	openId: _this.openId,
					// 	avatarUrl: res['fileID'],
					// 	nickName: _this.nicKname1,
					// }, {
					// 	showLoading: true
					// });
					// if (res.code == 200) {					
					// 	uni.setStorageSync(
					// 		'userInfo',
					// 		res.data);
					// 	uni.showToast({
					// 		icon: "success",
					// 		title: "保存成功",
					// 		duration: 2000
					// 	})
					// 	uni.switchTab({
					// 		url: '/pages/index/index'
					// 	});
					// } else {
					// 	uni.showToast({
					// 		icon: "success",
					// 		title: "保存失败",
					// 		duration: 2000
					// 	})
					// }
				})
			}
		}
	}
</script>


<style>
</style>
