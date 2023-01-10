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
				<u-avatar :src="userInfo.avatar" size="140"></u-avatar>
			</view>
			<view class="u-flex-1">
				<view class="u-font-18 u-p-b-20" v-if="!islogin" @click="login">登录</view>
				<view class="u-font-18 u-p-b-20" v-if="islogin">{{userInfo.nickName}}</view>
				<view class="u-font-14 u-tips-color" v-if="islogin">功德:{{userInfo.num}}</view>
			</view>
			<!-- <view class="u-m-l-10 u-p-10">
				<u-icon name="scan" color="#969799" size="28"></u-icon>
			</view> -->
			<view class="u-m-l-10 u-p-10">
				<u-icon name="arrow-right" color="#969799" size="28"></u-icon>
			</view>
		</view>

		<view class="u-m-t-20" v-if="islogin">
			<u-cell-group v-if="islogin">
				<u-cell-item  v-if="islogin" icon="share" title="分享   +5000功德" open-type="share"></u-cell-item>
				<!-- <u-image src="/static/imgs/yjfx.png" width="110" height="110"></u-image> -->
				<!-- <u-button id="shareBtn" open-type="share" ></u-button> -->
				<view class="share" v-if="islogin">
					<button  id="shareBtn" data-name="shareBtn" open-type="share">转发</button>
				</view>
			</u-cell-group>
		</view>

		<view class="u-m-t-20">
			<u-cell-group>
				<!-- <u-cell-item icon="star" @Clik="goIndex()" title="我的邀请"></u-cell-item> -->
				<!-- 	<u-cell-item icon="photo" title="相册"></u-cell-item>
				<u-cell-item icon="coupon" title="卡券"></u-cell-item>
				<u-cell-item icon="heart" title="关注"></u-cell-item> -->
			</u-cell-group>
		</view>

		<!-- <view class="u-m-t-20">
			<u-cell-group>
				<u-cell-item icon="setting" title="设置"></u-cell-item>
			</u-cell-group>
		</view> -->
	</view>
</template>

<script>
	import {
		request,
	} from '@/common/js/request'
	var _this;
	export default {
		data() {
			return {
				pic: 'https://uviewui.com/common/logo.png',
				show: true,
				userInfo: "",
				islogin: false
			}
		},
		onShareAppMessage(res) {
			if (res.from === 'button') { // 来自页面内分享按钮
				// console.log(222, res)
				// button 按钮分享
				let obj = res.target.dataset.obj // 获取 button 组件 自定义的data-obj值
				return {
					title: '快来和我一起积累功德吧', // 标题
					imageUrl: '/static/image/gongde2.jpg', // 封面图
					path: '/pages/index/index?datatype=' + _this.userInfo.openId // 地址以及参数 
				};
			
			} else {
				return {
					title: '自定义分享标题',
					path: '/pages/index/index'
				}
			}
		},
		onLoad() {
			_this = this;
			_this.userInfo = uni.getStorageSync('userInfo')
			if (_this.userInfo) {
				_this.islogin = true;
			}
		},
		onShow() {
			_this = this;
			if (!_this.userInfo) {
				uni.showToast({
					icon: "error",
					title: "请先登录",
					duration: 2000
				})
			} else {
				_this.getuserinfo();

				_this.userInfo = uni.getStorageSync('userInfo')
				// console.log(111,_this.userInfo)
				if (_this.userInfo) {
					_this.islogin = true;
				}
			}
		},
		methods: {
			islogin1() {
				_this.userInfo = uni.getStorageSync('userInfo')
				if (!_this.userInfo) {
					return false;
				} else {
					if (_this.userInfo.nickName == "微信用户" || !_this.userInfo.nickName) {
						uni.navigateTo({
							url: "/pagesA/user/setuser?openid=" + _this.userInfo.openId
						})
					} else {
						return true;
					}
				}
			},
			async getuserinfo() {
				const res = await request('tihuo', 'getuserinfo', {
					openId: _this.userInfo.openId
				}, {
					showLoading: false
				});
				if (res.code == 200) {
					_this.userInfo = res.data[0];
					uni.setStorageSync(
						'userInfo',
						res.data[0]);
				}
			},
			goIndex() {
				uni.navigateTo({
					url: "/pages/index/yaoqing"
				})
			},
			login() {
				uni.getUserProfile({
					lang: 'zh_CN',
					desc: '用于完善会员资料',
					success: userInfo => {
						uni.login({
							provider: 'weixin',
							success: loginInfo => {
								uniCloud.callFunction({
									name: 'tihuo',
									data: {
										operation: 'loginnew',
										data: {
											code: {
												code: loginInfo.code
											},
											data: {
												userInfo: userInfo.userInfo,
												userid: _this.userid
											}
										}
									}
								}).then(res => {
									if (res.result.code == 200) {
										_this.userInfo = res.result.data;
										_this.islogin = true;
										uni.setStorageSync(
											'userInfo',
											res.result.data);
										_this.islogin1();
										uni.showToast({
											icon: "none",
											title: "登陆成功",
											duration: 1000
										});
										return;
									}
								}).catch((err) => {
									uni.showToast({
										icon: "none",
										title: err
											.toString()
											.replace(
												"Error:",
												""),
										duration: 2000
									});
								})
							}
						});
					},
					fail: err => {
						console.log(err, 'err')
					}
				});
			}
		}
	}
</script>

<style lang="scss">
	page {
		background-color: #ededed;
	}

	.share {
		width: 100%;
		height: 100rpx;
		// border-radius: 50%;
		position: absolute;
		top: 10%;
		// background-color: red;
		right: 0rpx;

		#shareBtn {
			width: 100%;
			height: 100%;
			position: absolute;
			top: 0;
			left: 0;
			opacity: 0;
		}
	}

	.camera {
		width: 54px;
		height: 44px;

		&:active {
			background-color: #ededed;
		}
	}

	.user-box {
		background-color: #fff;
	}
</style>
