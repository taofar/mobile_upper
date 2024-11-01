<template>
	<view class="content">
		<uni-row v-for="(item,index) in configList" class="config-item">
			<uni-col :span="14" :offset="1">
				<h5>{{item.title}}</h5>
				<p>{{item.desc}}</p>
			</uni-col>
			<uni-col :span="6"><uni-number-box v-model="item.value" :max="item.max" background="#2979FF" color="#fff" /></uni-col>
			<uni-col :span="2">
				<uni-icons type="gear-filled" color="#007AFF" size="28" @click="onConfigClick(item)"></uni-icons>
			</uni-col>
		</uni-row>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				configList:[]
			}
		},
		created() {
			let config = uni.getStorageSync('swiftHelperGlobalConfig').config;
			if (config) {
				this.configList = config;
			}
			uni.$on('applicationUpdated', (e)=>{
				let confi = uni.getStorageSync('swiftHelperGlobalConfig').config;
				if (confi) {
					this.configList = confi;
				}
			});
		},
		onLoad() {
			uni.$on('onRecvConfigData', item=>{
				console.log(item);
				this.configList.filter(e=>e.key == item.key)[0].value = item.value;
			});
		},
		onShow() {
			uni.$emit('bluetoothSendString', 'xp config show\r\n');
		},
		methods: {
			onConfigClick(item) {
				let cmd = 'xp config set ' + item.key + '=' + item.value + '\r\n';
				console.log(cmd);
				uni.$emit('bluetoothSendString', cmd);
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		height: 99%;
		width: 99%;
		position: absolute;
		flex-direction: column;
		align-items: center;
		justify-content: flex-start;
	}
	.config-item {
		width: 100%;
		display: flex;
		flex-direction: row;
		background-color: rgba(0, 122, 255, .1);
		justify-content: center;
		align-items: center;
		align-content: center;
		/* border: #007AFF solid 2rpx; */
		border-radius: 10rpx;
		margin-top: 10rpx;
		padding: 20rpx 0;
	}
	.config-item p {
		font-size: 12px;
	}
</style>
