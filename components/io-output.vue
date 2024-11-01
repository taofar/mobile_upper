<template>
	<view>
		<view class="output-io-list-layout">
			<view class="output-io-item" v-for="(item, i) in inputIoAlias" :class="item.touched ? 'output-io-touched' : ''"
				@touchstart="onTouchStart(item)"
				@touchend="onTouchEnd"
				@touchmove="onTouchMove"
				@click="ioItemClick(item)">
				<p>{{item.id}}</p>
				<p>{{item.alias}}</p>
			</view>
		</view>
		<uni-popup ref="inputDialog" type="dialog">
			<uni-popup-dialog ref="inputClose"  mode="input" :title="'输出IO'+currentEditItem.id+'功能别名'"
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
				inputAlias: '',
				currentEditItem: {},
				aliasBit: 0,
				inputIoAlias: [],
				touchTime:0,
				touchTimeout: 0
			};
		},
		created() {
			let config = uni.getStorageSync('swiftHelperGlobalConfig').status.output;
			if (config) {
				this.inputIoAlias = config;
			}
			uni.$on('applicationUpdated', (e)=>{
				let config = uni.getStorageSync('swiftHelperGlobalConfig').status.output;
				if (config) {
					this.inputIoAlias = config;
				}
			});
		},
		mounted() {
			
		},
		methods: {
			ioItemClick(item) {
				if (item.touched) {
					uni.$emit('bluetoothSendString', item.closeCmd + '\r\n');
					console.log(item.closeCmd + '\r\n');
					item.touched = false;
				} else {
					uni.$emit('bluetoothSendString', item.openCmd + '\r\n');
					console.log(item.openCmd + '\r\n');
					item.touched = true;
				}
			},
			dialogInputConfirm(e) {
				this.currentEditItem.alias = e;
				console.log(e);
				
				let config = uni.getStorageSync('swiftHelperGlobalConfig');
				if (config) {
					config.status.output = this.inputIoAlias;
					uni.setStorageSync('swiftHelperGlobalConfig', config);
				}
				// this.currentEditItem.openCmd = item.openCmd;
				// this.currentEditItem.closeCmd = item.closeCmd;
				// 将改动后的值保存
			},
			onTouchStart(item) {
				this.touchTime = new Date().getTime();
				// console.log(this.touchTime);
				this.touchTimeout = setTimeout(()=>{
					this.inputAlias = item.alias;
					this.currentEditItem = item;
					console.log("longtouch");
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
	.output-io-list-layout {
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
	.output-io-item {
		/* // border: #000000 solid 2rpx; */
		display: flex;
		justify-content: space-between;
		width: 90%;
		min-height: 3%;
		margin-left: 0%;
		/* margin-right: 15%; */
		border-radius: 20rpx;
		padding-left: 5%;
		padding-right: 5%;
		background-color: rgba(0, 122, 255, 0.1);
	}
	.output-io-touched {
		background-color: #007AFF;
		color: #FFFFFF;
	}
</style>
