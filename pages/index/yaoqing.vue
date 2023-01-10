<template>
	<view class="container">
		<view class="rank-item" v-if="userInfo" style="position: relative; left: 0rpx; background-color: #eeeeee;">
			<!--头像-->
			<view class="rank-img">
				<image :src="userInfo.avatar"></image>
			</view>
			<!--展示昵称,以及贡献值-->
			<view>
				<view class="rank-name" style="color: #550000;">{{userInfo.nickName}}</view>
				<view class="rank-price">功德：{{userInfo.num}}次</view>
			</view>
			<!--排名-->
			<view class="rank-uv">
				<text>排名：第 {{mingci+1}} 名</text>
				<!-- <view class="rank-paiming">共邀请了：{{list.length}}名</view> -->
				<text style="margin-top: 70rpx; font-size: 28rpx; color: darkcyan;" class="rank-paiming">共邀请了：{{list.length}} 名</text>
			</view>
		</view>

		<view v-for="(item,index) in list" :key="index" class="rank_block">
			<view class="rank-item">
				<!--头像-->
				<view class="rank-img">
					<image :src="item.avatar"></image>
				</view>
				<!--展示昵称,以及贡献值-->
				<block>
					<view class="rank-name">{{item.nickName}}</view>
					<view class="rank-price">功德：{{item.num}}次</view>
				</block>
				<!--排名-->
				<view class="rank-uv">
					<text>第 {{index+1}} 名</text>
					<!-- <image src="/img/rank"{{index}}".png" v-if="{{index<=3}}" /> -->
				</view>
			</view>
		</view>
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
				list: "",
				userInfo: "",
				mingci: ""
			};
		},
		onShow() {
			_this = this;
			_this.userInfo = uni.getStorageSync('userInfo')
			if (_this.userInfo) {
				_this.islogin = true;
			}
			if (!_this.userInfo) {
				uni.showToast({
					icon: "error",
					title: "请先登录",
					duration: 2000
				})
			} else {
				_this.getuserinfo();
				_this.getuserpaiming();
				_this.userInfo = uni.getStorageSync('userInfo')
				if (_this.userInfo) {
					_this.islogin = true;
				}
			}
		},
		onLoad() {
			_this = this;
			_this.getuserpaiming();
			_this.userInfo = uni.getStorageSync('userInfo')
			if (_this.userInfo) {
				_this.islogin = true;
			}
		},
		methods: {
			async getuserinfo() {
				const res = await request('tihuo', 'getuserinfo', {
					openId: _this.userInfo.openId
				}, {
					showLoading: false
				});
				if (res.code == 200) {
					_this.userInfo=res.data[0];
					uni.setStorageSync(
						'userInfo',
						res.data[0]);
				}
			},
			async getuserpaiming() {
				const res = await request('tihuo', 'getmyuser', {
					openId: _this.userInfo.openId
				}, {
					showLoading: false
				});
				if (res.code == 200) {

					_this.list = res.data;
					_this.mingci = _this.list.findIndex(data => data.openId === _this.userInfo.openId);
				}
			}
		}
	};
</script>

<style>
	page {
		background-color: #fbf9fe;
	}
	.container {
		width: 100%;
		margin-bottom: 80rpx;
		margin-top: 10rpx;
		display: flex;
		flex-direction: column;
		justify-content: flex-end;
	}

	.rank-item {
		height: 100%;
		width: 100%;
		background: #fff;
		margin-bottom: 8rpx;
		padding: 20rpx 20rpx 20rpx 50rpx;
		box-sizing: border-box;
		position: relative;

	}

	.rank-item .rank-img {
		width: 100rpx;
		height: 100rpx;
		float: left;
		margin-right: 50rpx;
		position: relative;
	}

	.rank-item .rank-img image {
		position: absolute;
		border-radius: 50%;
		width: 100rpx;
		height: 100rpx;
		top: 0;
		left: 0;
	}

	.rank-item .rank-name {
		font-size: 32rpx;
		height: 50rpx;
		line-height: 50rpx;
		color: #4e5b65;
		font-weight: bold;
	}

	.rank-item .rank-full {
		font-size: 32rpx;
		height: 100rpx;
		line-height: 100rpx;
		color: #4e5b65;
	}

	.rank-item .rank-price {
		height: 40rpx;
		line-height: 40rpx;
		margin-top: 10rpx;
		font-size: 34rpx;
		top:0rpx;
		color: #550000;
	}
.rank-paiming {
		height: 40rpx;
		line-height: 40rpx;
		margin-top: 10rpx;
		font-size: 14rpx;
		top:0rpx;
		color: #550000;
	}
	.rank-item .rank-uv text {
		font-size: 38rpx;
		position: absolute;
		height: 100rpx;
		line-height: 100rpx;
		bottom: 20rpx;
		right: 20rpx;
		color: #777;
	}

	.rank-item .rank-uv image {
		position: absolute;
		width: 100rpx;
		height: 100rpx;
		bottom: 20rpx;
		right: 20rpx;
	}

	.rank_block {
		margin-top: 10rpx;
	}
</style>
