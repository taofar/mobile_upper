<template>
	<view>
		<view class="input-io-list-layout">
			<view class="input-io-item" v-for="(item, i) in inputIoAlias" :class="item.touched ? 'input-io-touched' : ''"
				@touchstart="onTouchStart(item)"
				@touchend="onTouchEnd"
				@touchmove="onTouchMove">
				<p>{{item.id}}</p>
				<p>{{item.alias}}</p>
			</view>
		</view>
		<!-- <view class="bitmap-toolbar">
			<view class="bitmap-selectbar"></view>
			<view class="bitmap-menubar"></view>
		</view> -->
		<uni-popup ref="inputDialog" type="dialog">
			<uni-popup-dialog ref="inputClose"  mode="input" :title="'输入IO'+currentEditItem.id+'功能别名'"
				@confirm="dialogInputConfirm" :value="inputAlias"></uni-popup-dialog>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		name:"io-watch",
		data() {
			return {
				timer:null,
				inputIoValue: 0xFFFFFFFF,
				inputAlias: '',
				currentEditItem: {},
				aliasBit: 0,
				inputIoAlias: []
			};
		},
		created() {
			let config = uni.getStorageSync('swiftHelperGlobalConfig').status.input;
			if (config) {
				console.log('====================');
				this.inputIoAlias = config;
			}
			uni.$on('applicationUpdated', (e)=>{
				let config = uni.getStorageSync('swiftHelperGlobalConfig').status.input;
				if (config) {
					this.inputIoAlias = config;
				}
			});
		},
		props: {
			value: {
				type:Number,
				default:()=>0
			},
			bitWidth: {
				type:Number,
				default:()=>24
			}
		},
		mounted() {
			this.inputIoValue = this.value;
			let width = this.bitWidth > 32 ? 32 : this.bitWidth;
			for (let i = 0; i < width; i ++) {
				this.inputIoAlias[i].touched = (this.inputIoValue & (1 << i)) ? false : true;
			}
		},
		watch: {
			'value': function(v) {
				this.inputIoValue = v;
				let width = this.bitWidth > 32 ? 32 : this.bitWidth;
				for (let i = 0; i < this.bitWidth; i ++) {
					this.inputIoAlias[i].touched = (this.inputIoValue & (1 << i)) ? false : true;
				}
			}
		},
		methods: {
			dialogInputConfirm(alias) {
				// console.log(alias);
				this.currentEditItem.alias = alias;
				// 将改动后的值保存
				let config = uni.getStorageSync('swiftHelperGlobalConfig');
				if (config) {
					config.status.input = this.inputIoAlias;
					uni.setStorageSync('swiftHelperGlobalConfig', config);
				}
			},
			onTouchStart(item) {
				this.touchTime = new Date().getTime();
				// console.log(this.touchTime);
				this.touchTimeout = setTimeout(()=>{
					this.inputAlias = item.alias;
					this.currentEditItem = item;
					this.$refs.inputDialog.open();
				}, 500);
			},
			onTouchMove() {
				clearTimeout(this.touchTimeout);
			},
			onTouchEnd() {
				if (new Date().getTime() - this.touchTime > 500) {
					return ;
				}
				clearTimeout(this.touchTimeout);
			}
		}
	}
</script>

<style>
	.input-io-list-layout {
		display: flex;
		height: 100%;
		width: 100%;
		position: absolute;
		flex-direction: column;
		align-items: center;
		border-radius: 20rpx;
		justify-content: space-between !important;
		left: 0;
		top: 0;
	}
	.input-io-item {
		/* // border: #000000 solid 2rpx; */
		display: flex;
		justify-content: space-between;
		width: 90%;
		min-height: 3%;
		margin-left: 0%;
		margin-right: 0%;
		border-radius: 20rpx;
		padding-left: 5%;
		padding-right: 5%;
		background-color: rgba(0, 122, 255, 0.1);
	}
	.input-io-touched {
		background-color: #007AFF;
		color: #FFFFFF;
	}
	.bitmap-toolbar {
		width: 13%;
		height: 100%;
		right: 0;
		top: 0;
		border-radius: 20rpx;
		background-color: rgba(0, 122, 255, 0.1);
		position: absolute;
	}
</style>
