<template>
	<view class="content">
		<form>

			<view class="cu-form-group">
				<view class="title">预约清单</view>
			</view>
			<view class="" style="position: relative;z-index: 10000;">
				<uni-datetime-picker type="date" :clear-icon="false" v-model="single" @maskClick="maskClick" />
			</view>
			<view v-for="(item,index) in orderGoods " v-bind:key="index" class="cu-form-group">
				<image class="avator" :src="baseUrl+item.yimiaoPhoto" mode=""></image>
				<view class="title" style="width: 75%;">
					<view style="margin-top: -50rpx;">{{item.yimiaoName}}</view>

				</view>
			</view>

		</form>
		<view class="padding" style="text-align: center;">
			<button @tap="onSubmitTap()" class="bg-red lg">确认预约</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				single: '',
				user: {}, //登录用户
				orderGoods: [], //展示数据
				maxNewMouey: 0, //总价格
				yimiaoOrderPaymentTypes: 1, //是什么支付类型
				zhi: [{
						id: 1,
						val: "余额"
					},
					{
						id: 2,
						val: "积分"
					},
				],
				zhekou: 1, //折扣
			}
		},
		computed: {
			baseUrl() {
				return this.$base.url;
			},
		},
		async onLoad(options) {
			// 获取订单信息，座位信息
			this.orderGoods = uni.getStorageSync('orderGoods');
			if (this.orderGoods.length > 0) {
				for (let i = 0; i < this.orderGoods.length; i++) {
					this.maxNewMouey = this.maxNewMouey + parseFloat(this.orderGoods[i].yimiaoNewMoney) * this
						.orderGoods[i].buyNumber
				}
			}
			uni.removeStorageSync("orderGoods")
		},
		async onShow() {
			let _this = this
			let table = uni.getStorageSync("nowTable");
			let userRes = await _this.$api.session(table)
			_this.user = userRes.data

			let huiyuandengjiTypesRes = await _this.$api.page("dictionary", {
				dicCode: "huiyuandengji_types",
				dicName: "会员等级类型",
				codeIndexStart: _this.user.huiyuandengjiTypes,
				codeIndexEnd: _this.user.huiyuandengjiTypes,
			})
			if (huiyuandengjiTypesRes.data.list.length > 0) {
				_this.zhekou = Number(huiyuandengjiTypesRes.data.list[0].beizhu);
			}


		},
		methods: {
			maskClick(e) {
				console.log('maskClick事件:', e);
			},

			async onSubmitTap() {
				let _this = this;
				let table = uni.getStorageSync("nowTable");
				uni.showModal({
					title: '提示',
					content: '是否确认预约',
					success: async function(res) {
						if (res.confirm) {
							let data = {
								yimiaos: JSON.stringify(_this.orderGoods),
								yonghuId: _this.user.id,
								// yimiaoOrderPaymentTypes:  _this.yimiaoOrderPaymentTypes,
								yimiaoOrderTime: _this.single,
							}
							await _this.$api.requestConditionDataGet('yimiaoOrder', 'order', null, data);
							_this.$utils.jump('/pages/yimiaoOrder/list');
						}
					}
				});
			},
		}
	}
</script>

<style lang="scss">
	.avator {
		width: 150upx;
		height: 150upx;
		margin: 20upx 0;
	}
</style>
