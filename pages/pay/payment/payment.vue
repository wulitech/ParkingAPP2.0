<template>
	<view>
		<view class="block">
			<view class="content">
				<view class="orderinfo">
					<view class="row">
						<view class="nominal">订单名称:</view>
						<view class="text">{{orderName}}</view>
					</view>
					<view class="row">
						<view class="nominal">订单金额:</view>
						<view class="text">{{amount}}元</view>
					</view>
				</view>
			</view>
		</view>
		<view class="block">
			<view class="title">
				选择支付方式
			</view>
			<view class="content">
				<view class="pay-list">
					<view class="row" @tap="paytype='alipay'">
						<view class="left">
							<image src="/static/img/alipay.png"></image>
						</view>
						<view class="center">
							支付宝支付
						</view>
						<view class="right">
							<radio :checked="paytype=='alipay'" color="#f06c7a" />
						</view>
					</view>
					<view class="row" @tap="paytype='wxpay'">
						<view class="left">
							<image src="/static/img/wxpay.png"></image>
						</view>
						<view class="center">
							微信支付
						</view>
						<view class="right">
							<radio :checked="paytype=='wxpay'" color="#f06c7a" />
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="pay">
			<view class="btn" @tap="doDeposit">立即支付</view>
			<view class="tis">
				点击立即支付，即代表您同意<view class="terms">
					《条款协议》
				</view>
			</view>
		</view>
		<view class="qrimg" style="visibility: hidden;">
			<tki-qrcode ref="qrcode" :val="order_info.orderId" :size="size" :unit="unit" :icon="icon" :iconSize="iconsize"
			 @result="qrR" />
		</view>
	</view>
</template>

<script>
	import tkiQrcode from '@/components/tki-qrcode/tki-qrcode.vue'

	export default {
		components: {
			tkiQrcode
		},
		data() {
			return {
				order_info: {},
				amount: 0,
				orderName: '',
				paytype: 'alipay', //支付类型
				size: 200, // 二维码大小
				unit: 'upx', // 单位
				icon: 'static/logo/logo.png', // 二维码图标
				iconsize: 40, // 二维码图标大小
				loadMake: false, // 组件加载完成后自动生成二维码
				src: '', // 二维码生成后的图片地址或base64
			};
		},
		onLoad(e) {
			this.order_info = JSON.parse(e.order_info);
			this.amount = parseFloat(this.order_info.price).toFixed(2);
			this.orderName = this.order_info.parking_name;
		},
		methods: {
			doDeposit() {
				var order_info = JSON.stringify(this.order_info)
				//模板模拟支付，实际应用请调起微信/支付宝
				uni.showLoading({
					title: '支付中...'
				});
				setTimeout(() => {
					this.creatQrcode();
				},600)
			},
			creatQrcode() {
				this.$refs.qrcode._makeCode()
			},
			qrR(res) {
				var order_info1 = JSON.stringify(this.order_info)
				this.order_info.qr_code = res;
				var order_info = JSON.stringify(this.order_info)
				/* 这边调后台接口支付接口,data为order_info,回调函数执行一下内容; */
				uni.request({
					url: this.$api + '/order/pay',
					data: order_info,
					method: 'POST',
					dataType: 'json',
					success: (res) => {
						if (res.data.code != 200) {
							uni.hideLoading();
							uni.showToast({
								title: '支付失败，请稍候重试'
							});
						} else {
							uni.hideLoading();
							uni.showToast({
								title: '支付成功'
							});

							uni.redirectTo({
								url: '../../pay/success/success?order_info=' + order_info1
							});

						}
					}
				});

			},

		}
	}
</script>

<style lang="scss">
	.block {
		width: 94%;
		padding: 0 3% 40upx 3%;

		.title {
			width: 100%;
			font-size: 34upx;
		}

		.content {
			.orderinfo {
				width: 100%;
				border-bottom: solid 1upx #eee;

				.row {
					width: 100%;
					height: 90upx;
					display: flex;
					align-items: center;

					.nominal {
						flex-shrink: 0;
						font-size: 32upx;
						color: #7d7d7d;
					}

					.text {
						width: 70%;
						margin-left: 10upx;
						overflow: hidden;
						text-overflow: ellipsis;
						white-space: nowrap;
						font-size: 32upx;
					}
				}
			}

			.pay-list {
				width: 100%;
				border-bottom: solid 1upx #eee;

				.row {
					width: 100%;
					height: 120upx;
					display: flex;
					align-items: center;

					.left {
						width: 100upx;
						flex-shrink: 0;
						display: flex;
						align-items: center;

						image {
							width: 80upx;
							height: 80upx;
						}
					}

					.center {
						width: 100%;
						font-size: 30upx;
					}

					.right {
						width: 100upx;
						flex-shrink: 0;
						display: flex;
						justify-content: flex-end;
					}
				}
			}
		}
	}

	.pay {
		margin-top: 20upx;
		width: 100%;
		display: flex;
		justify-content: center;
		flex-wrap: wrap;

		.btn {
			width: 70%;
			height: 80upx;
			border-radius: 80upx;
			display: flex;
			justify-content: center;
			align-items: center;
			color: #fff;
			background-color: #f06c7a;
			box-shadow: 0upx 5upx 10upx rgba(0, 0, 0, 0.2);
		}

		.tis {
			margin-top: 10upx;
			width: 100%;
			font-size: 24upx;
			display: flex;
			justify-content: center;
			align-items: baseline;
			color: #999;

			.terms {
				color: #5a9ef7;
			}
		}
	}
</style>
