<template>
	<view class="content">
		<view style="height: 100rpx;" class=""></view>
		<view style="height: 300rpx;" class=""></view>
		<view class="list">
			<view v-for="(item,index) of list" :key="index" class="list-item">
				{{item}}
			</view>
		</view>
		<image @click="woodFishHandle" :class="status === 1?'wooden_fish_act':''" class="wooden_fish"
			src="@/static/muyu.png" mode=""></image>
		<view class="auto">
			<text style="color: white;text-align: center; font-size:21pt">本次功德：{{num}}</text>
			<!-- <button></button> -->
			<!-- <button @click="auto" v-if="status === 0">自动功德</button> -->
			<!-- <button @click="close" v-if="status === 1">关闭自动功德</button> -->
		</view>
		<u-popup mode="left" :show="show" @close="setting_close" @open="setting_open">
			<view class="setting-block">
				<input class="inp" type="text" placeholder="例:功德 +1" v-model="text" name="" id="">
			</view>
		</u-popup>
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
				title: 'Hello',
				audio: require("@/static/audio.mp3"),
				list: [],
				timer: null, //定时器
				status: 0, //状态
				show: false,
				text: "功德 +1",
				num: 0,
				userid: ""
			}
		},
		onHide() {
			_this = this;
			_this.close();
			if (_this.num != 0) {
				uniCloud.callFunction({
					name: 'tihuo',
					data: {
						operation: 'upgongde',
						data: {
							openId: _this.userInfo.openId,
							num: _this.num
						}
					}
				}).then(res => {
					// console.log(11111, res)
					if (res.result.code == 200) {
						_this.userInfo.num += _this.num;
						_this.num = 0;
						// console.log(88888, _this.userInfo)
						uni.setStorageSync(
							'userInfo',
							_this.userInfo);
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
		},
		onLoad(options) {
			_this = this;
			_this.userInfo = uni.getStorageSync('userInfo')
			// if (_this.userInfo) {
			// 	// _this.autologin();
			// } else {
			// 	_this.userid = options.datatype;
			// }
			if (!options) {
				_this.userid = "";
			} else {
				_this.userid = options.datatype;
			}
		},
		methods: {
			autologin() {
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
										uni.setStorageSync(
											'userInfo',
											res.result.data);
										uni.showToast({
											icon: "none",
											title: "登陆成功",
											duration: 1000
										});
										_this.islogin();
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
			},
			islogin() {
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
			woodFishHandle() {
				if (!_this.islogin()) {
					_this.autologin();
					return;
				} else {
					this.list.push(_this.text)
					const innerAudioContext = uni.createInnerAudioContext();
					innerAudioContext.autoplay = true;
					innerAudioContext.src = this.audio;
					innerAudioContext.onPlay(() => {
						// console.log('开始播放');
					});
					this.num++;
				}
			},
			auto() {
				let _this = this
				_this.status = 1
				_this.timer = setInterval(function() {
					_this.woodFishHandle()
				}, 1000)
			},
			close() {
				let _this = this
				_this.status = 0
				clearInterval(_this.timer)
			},
			setting_open() {
				// console.log('open');
			},
			setting_close() {
				this.show = false
				// console.log('close');
			}
		}
	}
</script>

<style>
	@keyframes text-animation {

		from {
			top: 0;
		}

		to {
			top: -300rpx;
			opacity: .0;
		}

	}

	@keyframes fish-animation {

		from {
			transform: scale(1);
		}

		to {
			transform: scale(1.2);
		}

	}

	page {
		background-color: #202020;
	}

	.content {
		position: relative;
		height: 100vh;
	}

	.wooden_fish {
		width: 260rpx;
		height: 199rpx;
		position: absolute;
		left: 50%;
		top: 50%;
		margin-left: -130rpx;
		margin-top: -99.5rpx;
	}

	.wooden_fish_act {
		animation: fish-animation linear .1s infinite alternate;
	}

	.wooden_fish:active {
		transform: scale(1.2);
	}

	.list {
		text-align: center;
		color: #fff;
		position: relative;
		display: flex;
		justify-content: center;

	}

	.list-item {
		position: absolute;
		animation: text-animation linear 1.5s forwards;
	}

	.auto {
		position: fixed;
		bottom: 100rpx;
		width: 100%;
		/* display: flex; */
		text-align: center;
	}

	.tools {
		height: 100rpx;
		padding-left: 50rpx;
		box-sizing: border-box;
	}

	.icon-setting {
		width: 50rpx;
		height: 50rpx;
	}

	.setting-block {
		width: 500rpx;
		padding: 20rpx;
		box-sizing: border-box;
	}

	.inp {
		border: 1px solid #eee;
		border-radius: 20rpx;
		height: 72rpx;
		padding: 6rpx 20rpx;
		font-size: 28rpx;
	}
</style>
