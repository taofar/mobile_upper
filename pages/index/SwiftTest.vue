<template>
	<view>
		<uni-grid :column="3" :highlight="true">
			<uni-grid-item v-for="(item, index) in list" :index="index" :key="index">
				<view class="grid-item-box" :class="{'grid-item-success': item.tested && item.testOK,'grid-item-fail': item.tested && !item.testOK}" @click="onClick(item)">
					<h4>{{item.title}}</h4>
					<!-- <text class="text"></text> -->
				</view>
			</uni-grid-item>
		</uni-grid>
		<uni-popup ref="testPopup" background-color="#fff">
			<view style="width: 100%;">
				<uni-card :title="testItem.title" :is-shadow="false">
					<p class="test-item-desc">{{testItem.desc}}</p>
					<view>
						<button class="test-item-button" v-for="(item, index) in testItem.cmdList" type="primary" @click="testButtonClick(item)">{{item.name}}</button>
					</view>
					<p v-if="testItem.needAck" class="test-item-resdata">设备响应数据 : {{testItem.resData}}</p>
					<view>
					<image v-if="testItem.ackLink && image_url.length > 50" mode="aspectFit" :src="image_url"></image>
					<textarea v-if="testItem.ackLink && image_url.length > 50" :value="image_url"></textarea>
					</view>
					<uni-collapse >
						<uni-collapse-item title="异常修复指南" :show-animation="true">
							<view>
								<p v-for="(item,index) in testItem.help" class="text">{{(index+1) + '. ' + item}}</p>
							</view>
						</uni-collapse-item>
					</uni-collapse>
					<uni-row class="test-item-passBtn">
						<uni-col :span="11"><button type="warn" @click="testPass(false, testItem)">测试不通过</button></uni-col>
						<uni-col :span="11" :offset="2"><button type="primary" @click="testPass(true, testItem)">测试通过</button></uni-col>
					</uni-row>
				</uni-card>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '',
				dynamicList: [],
				list: [],
				tested:[false],
				testOK:[false],
				image_url: "",
				testItem: {
					resData: ""
				}
			}
		},
		created() {
			let config = uni.getStorageSync('swiftHelperGlobalConfig').test;
			if (config) {
				this.list = config;
			}
			uni.$on('applicationUpdated', (e)=>{
				let config = uni.getStorageSync('swiftHelperGlobalConfig').test;
				if (config) {
					this.list = config;
				}
			});
		},
		onLoad() {
			uni.$on('onRecvTestData', str=>{
				this.testItem.resData = str;
				this.image_url = "https://source.1kmxc.com/rk3399pro/" + str.trim();
				console.log(this.image_url);
				this.$forceUpdate();
			});
			for(let item in this.list) {
				this.list[item].id = item;
				this.list[item].tested = false;
				this.list[item].testOK = false;
				this.list[item].resData = "";
			}
		},
		methods: {
			onClick(item) {
				// this.list[item.id].tested = true;
				// console.log(item);
				this.testItem = item;
				this.$refs.testPopup.open('bottom');
				//this.$forceUpdate();
			},
			change(e) {
				let {
					index
				} = e.detail
				console.log(this.list[index].testOK);
				// this.list[index].testOK = false;
				// this.$set(this.list[index], 'testOK', false);
				this.list[index].badge && this.list[index].badge++

				uni.showToast({
					title: `点击第${index+1}个宫格`,
					icon: 'none'
				})
			},
			testButtonClick(item) {
				console.log(item.cmd);
				let cmd = item.cmd + '\r\n';
				uni.$emit('bluetoothSendString', cmd);
			},
			testPass(isPass, item) {
				// console.log(item);
				if (isPass) {
					this.testItem.tested = true;
					this.testItem.testOK = true;
				} else {
					this.testItem.tested = true;
					this.testItem.testOK = false;
				}
				this.$refs.testPopup.close();
				this.$forceUpdate();
			}
		}
	}
</script>

<style>

	.grid-item-box {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 15px 0;
	}
	
	.grid-item-success {
		background-color: rgba(0, 122, 255, 1);
	}
	.grid-item-fail {
		background-color: rgba(255, 90, 95, 1);
	}
	
	.test-item-desc {
		padding-top: 10rpx;
		padding-bottom: 20rpx;
	}
	
	.test-item-button {
		margin: 10rpx 0;
	}
	.test-item-resdata {
		border: #000000, solid 2rpx;
		box-shadow: 10rpx 10rpx 10rpx #B9B9B9;
		border-radius: 10rpx;
		margin: 20rpx 0;
		padding: 20rpx 20rpx;
		/* background-color: rgba($color: #007AFF, $alpha: 0.2); */
		background-color: rgba(0, 122, 255, .3);
	}
	
	.test-item-passBtn {
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		padding: 20rpx 0rpx;
	}

</style>
