<template>
	<view class="wrapper">
		<view class="device-area">
			<view class="device-cart">
				<view class="device-info">
					<view class="device-name">温度</view>
					<image class="device-logo" src="/static//temp.png"></image>
				</view>
				<view class="device-data">{{Temp}}℃</view>
			</view>
			
			<view class="device-cart">
				<view class="device-info">
					<view class="device-name">湿度</view>
					<image class="device-logo" src="/static//humi.png"></image>
				</view>
				<view class="device-data">{{Humi}}%</view>
			</view>
			
			<view class="device-cart">
				<view class="device-info">
					<view class="device-name">雾化器</view>
					<image class="device-logo" src="/static//dohumi.png"></image>
				</view>
				<switch @change="onSHISwitch" color="#5b7891" :checked="SHI"/>
			</view>
			
			<view class="device-cart">
				<view class="device-info">
					<view class="device-name">风扇</view>
					<image class="device-logo" src="/static///fan.png"></image>
				</view>
				<switch @change="onLedSwitch" color="#5b7891" :checked="FAN"/>
			</view>
			
			<view class="device-cart">
				<view class="device-info">
					<view class="device-name">制冷片</view>
					<image class="device-logo" src="/static//cool.png"></image>
				</view>
				<switch @change="onCOLSwitch" color="#5b7891" :checked="COL"/>
			</view>
			
			<view class="device-cart">
				<view class="device-info">
					<view class="device-name">加热片</view>
					<image class="device-logo" src="/static//warm.png"></image>
				</view>
				<switch @change="onHOTSwitch" color="#5b7891" :checked="HOT"/>
			</view>
		</view>
		
		<!-- <image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view> -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				Temp: 0,
				Humi: 0,
				FAN: false,
				SHI: false,
				HOT: false,
				COL: false,
				out: 0,
				sendsum: 0,
				sendfan: 0,
				sendshi: 0,
				sendhot: 0,
				sendcol: 0,
			}
		},
		onShow() {
			let that = this
			this.GetDatapoints()
			setInterval(function(){
				that.GetDatapoints()				
			},3000)
		},
		onLoad() {
			// uni.request({
			//     url: 'http://api.heclouds.com/devices/1063643708/datapoints?', //仅为示例，并非真实接口地址。
			//     header: {
			//         'api-key':'05GaCL8iWtgi4hgoZH=B4GxRG=8=' //自定义请求头信息
			//     },
			// 	method:'GET',
			//     success: (res) => {
			//         console.log(res.data);
			// 		console.log(res.data.data.datastreams[1].id,res.data.data.datastreams[1].datapoints[0].value);
			// 		console.log(res.data.data.datastreams[0].id,res.data.data.datastreams[0].datapoints[0].value);
			// 		this.Temp = res.data.data.datastreams[1].datapoints[0].value;
			// 		this.Humi = res.data.data.datastreams[0].datapoints[0].value;
			//     }
			// });
		},
		methods: {
			GetDatapoints:function(){
				uni.request({
				    url: 'http://api.heclouds.com/devices/1063643708/datapoints?', //仅为示例，并非真实接口地址。
				    header: {
				        'api-key':'Qxl2IZ82aKhot3QKab0nKEE0ddI=' //自定义请求头信息
				    },
					method:'GET',
				    success: (res) => {
				  //       console.log(res.data);
						// console.log(res.data.data.datastreams[1].id,res.data.data.datastreams[1].datapoints[0].value);
						// console.log(res.data.data.datastreams[0].id,res.data.data.datastreams[0].datapoints[0].value);
						this.Temp = res.data.data.datastreams[1].datapoints[0].value;
						this.Humi = res.data.data.datastreams[0].datapoints[0].value;
						this.out = res.data.data.datastreams[2].datapoints[0].value;
						this.out = (this.out);						
						this.FAN = Boolean(parseInt(this.out%2));
						this.SHI = Boolean((parseInt(this.out/2))%2);
						this.HOT = Boolean((parseInt(this.out/4))%2);
						this.COL = Boolean(parseInt(this.out/8));
						console.log("----------out----------");
						console.log(this.out);
						console.log("----------out----------");
						
						console.log("^^^^^^^^^^^^FAN^^^^^^^^^^^^");
						console.log(parseInt(this.out%2));
						console.log(this.FAN);
						
						console.log("^^^^^^^^^^^^SHI^^^^^^^^^^^^");
						console.log((parseInt(this.out/2))%2);
						console.log(this.SHI);
						
						console.log("^^^^^^^^^^^^HOT^^^^^^^^^^^^");
						console.log((parseInt(this.out/4))%2);
						console.log(this.HOT);
						
						console.log("^^^^^^^^^^^^COL^^^^^^^^^^^^");
						console.log(parseInt(this.out/8));
						console.log(this.COL);
				    }
				});
			},
			onLedSwitch(event){
				console.log(event.detail.value);
				//console.log(res);
				let sw1 = event.detail.value;
				if(sw1){
					this.sendfan = 1;
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("FAN on!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
				else{
					this.sendfan = 0,
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("FAN off!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
			},
			onSHISwitch(event){
				console.log(event.detail.value);
				//console.log(res.data.data.datastreams[0].datapoints[0].value);
				let sw2 = event.detail.value;
				if(sw2){
					this.sendshi = 1;
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("SHI on!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
				else{
					this.sendshi = 0;
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("SHI off!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
			},
			
			onHOTSwitch(event){
				console.log(event.detail.value);
				//console.log(res.data.data.datastreams[0].datapoints[0].value);
				let sw3 = event.detail.value;
				if(sw3){
					this.sendhot = 1;
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("HOT on!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
				else{
					this.sendhot = 0;
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("HOT off!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
			},
			
			onCOLSwitch(event){
				console.log(event.detail.value);
				//console.log(res.data.data.datastreams[0].datapoints[0].value);
				let sw4 = event.detail.value;
				if(sw4){
					this.sendcol = 1;
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("COL on!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
				else{
					this.sendcol = 0;
					this.sendsum = this.sendfan + 2*this.sendshi + 4*this.sendhot + 8*this.sendcol;
					uni.request({
					    url: 'http://api.heclouds.com/mqtt?topic=LED_SW', //仅为示例，并非真实接口地址。
					    header: {
					        'api-key':'=PE2rSiPSkGapmy0cwU=0OYUIVs=' //Master-key
					    },
						method:'POST',
						data: {"LED":this.sendsum},
					    success: (res) => {
					        console.log("COL off!");
							console.log("====================");
							console.log(this.sendsum);
							console.log("====================");
					    }
					});
				}
			}
		}
	}
</script>

<style>
	.wrapper {
		padding: 30rpx;
	}

	.device-area {
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap;
	}

	.device-cart {
		width: 320rpx;
		height: 150rpx;
		border-radius: 30rpx;
		margin-top: 40rpx;
		display: flex;
		justify-content: space-around;
		align-items: center;
		/* background-color: #8f8f94; */
		box-shadow: 0 0 15rpx #ccc;
	}

	.device-info {
		font-size: 20rpx;
		/* background-color: #8f8f94; */
	}
	.device-name {
		text-align: center;
		color: #6d6d6d;
	}
	.device-logo {
		width: 70rpx;
		height: 70rpx;
		margin-top: 10rpx;
	}
	.device-data {
		font-size: 50rpx;
		color: #6d6d6d;
		
	}
</style>
