<template>
	<view class="content">
		<!-- 画布，在调试的时候可以将隐藏样式先去掉；
		注意：不能用display:none;或者是opacity:0;之类的隐藏画布，
		这些在编译器上不显示，但是在真机运行时是会显示出来的。 -->
		<view class="canvas-box">
		  <canvas canvas-id='shareFriend' style='width:300px;height:240px;'></canvas>
		</view>
		<!-- 商品列表 -->
		<scroll-view class="listWrap" :scroll-y="true">
			<view class="goodsItem" v-for="(goods,index) in goodsList" :key="index">
				<view class="imgBox">
					<view class="chooseFlag" v-show="nowSelect == index">√</view>
					<image :src="goods.pic" mode="widthFix"></image>
				</view>
				<view class="goodsInfo">
					<view class="goddsName">
						{{goods.name}}
					</view>
					<view class="sale">
						原价：￥{{goods.sale}}
					</view>
					<view class="nowSale">
						现在只售：<text class="heightLight">￥</text><text class="heightLight textBig">{{goods.nowSale}}</text>
					</view>
				</view>
				<view class="chooseBtn" @click="chooseGoods(index)">
					选择
				</view>
			</view>
		</scroll-view>
		<!-- 分享按钮 -->
		<view class="shareBtnList">
			<button type="primary" size="mini" @click="openPanle">点击分享</button>
		</view>
		<!-- 分享面板 -->
		<view class="sharePlaneWrap" @click="closePanle" v-if="sharePlaneStatus">
			<view class="sharePlane" @click.stop>
				<view class="top">
					<button class="leftGoods" open-type="share">
						<image src="../../static/wechat-icon.png" mode="widthFix"></image>
						<view class="">
							分享链接
						</view>
					</button>
					<button class="rightPoster" @click="sharePoster">
						<image src="../../static/poster-icon.png" mode="widthFix"></image>
						<view class="">
							分享海报
						</view>
					</button>
				</view>
				<view class="btm">
					<button type="default" @click="closePanle">取消</button>
				</view>
			</view>
		</view>
		<!-- 图片预览面板 -->
		<view class="posterPreview" v-if="posterStatus" @click="posterStatus = false">
			<image :src="picTempUrl" mode="widthFix"></image>
			<button type="default" @click.stop="getPhotosAlbumAuth">保存海报</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				nowSelect:null,//当前商品
				sharePlaneStatus:false,
				goodsList:[
					{id:'1001',name:'崩沙西瓜',sale:'35',nowSale:'25',pic:'../../static/wetermelon.jpg'},
					{id:'1002',name:'甘肃葡萄',sale:'18',nowSale:'13',pic:'../../static/grape.jpg'},
					{id:'1003',name:'新疆哈密瓜',sale:'55',nowSale:'42',pic:'../../static/hamimelon.jpg'},
					{id:'1004',name:'爽口青枣',sale:'32',nowSale:'25',pic:'../../static/jujube.jpg'},
					{id:'1005',name:'奇异果',sale:'60',nowSale:'51',pic:'../../static/kiwi.jpg'},
					{id:'1006',name:'水晶密橙',sale:'15',nowSale:'12',pic:'../../static/orange.jpg'},
					{id:'1007',name:'水果萝卜',sale:'20',nowSale:'15',pic:'../../static/radish.jpg'},
					{id:'1008',name:'晶莹雪梨',sale:'30',nowSale:'23',pic:'../../static/pear.jpg'}
				],//商品列表
				picTempUrl:'',
				posterStatus:false,
			}
		},
		onLoad() {

		},
		/**
		   * 分享到微信好友
		   */
		onShareAppMessage(){
			return {
			  title: '限时特卖：'+this.goodsList[this.nowSelect].name,
			  path: 'pages/index/index?data=这边可以传一些ID啥的',
			  imageUrl: this.picTempUrl
			}
		},
		methods: {
			//选择商品
			chooseGoods(id){
				this.nowSelect = id;
			},
			//开启分享面板
			openPanle(){
				if(this.nowSelect == null){
					uni.showToast({
						icon:'none',
						title: '请选择商品！',
						duration: 2000
					});
					return false;
				}
				this.shareGoods();
				this.sharePlaneStatus =true;
			},
			//关闭分享面板
			closePanle(){
				this.sharePlaneStatus =false;
			},
			//分享商品
			shareGoods(){
				this.createShareGoods();
			},
			//分享海报
			sharePoster(){
				//那个二维码是后来想起来加上去有点丑，你们自己设计需要考虑二维码或者小程序码的位置
				this.posterStatus = true;
			},
			//生成微信临时图片(引用网络图片时需要将网络图片转换为微信临时本地地址)
			// handleNetImg(imagePath) {
			// 	return new Promise((resolve, reject) => {
			// 		uni.getImageInfo({
			// 			src: imagePath,
			// 			success: function (res) {
			// 				resolve(res);
			// 			}
			// 		});
			// 	});
			// },
			//创建画布
			createShareGoods(){
				uni.showLoading({
					title:'加载中...'
				})
				let selectGoods = this.goodsList[this.nowSelect];
				let bgdSrc ='../../static/poster1.png';
				let qrcode ='../../static/qrcode.png'
				//如果图片是网络图片需要调用该函数转为本地图片地址；例：
				// this.handleNetImg('https://test.oss-cn-shanghai.aliyuncs.com/test.png').then((res)=>{
				// 	console.log(res.path);//用这张图来生成bgdSrc
				//	bgdSrc =res.path;
				//	canvas操作...
				// ... 
				// ...
				// }).catch((errorMsg) => {
				//   uni.showToast({
				//     title: errorMsg,
				//     mask: true,
				//     icon: "none",
				//     duration: 10000
				//   })
				// })
				var ctx = uni.createCanvasContext('shareFriend', this);
				ctx.drawImage(bgdSrc, 0, 0, 300, 240); //画背景图
				ctx.drawImage(selectGoods.pic, 0, 0, 400, 400, 30, 45, 140, 140);//画商品图片
				ctx.drawImage(qrcode, 0, 0, 400, 400, 225, 5, 100, 100);//画二维码,这边可以变成小程序码
				//现价
				ctx.setFillStyle('#E80013');
				ctx.setFontSize(18);
				ctx.fillText('限时特价', 180, 100);
				ctx.setFillStyle('#E80013');
				ctx.setFontSize(15);
				ctx.fillText("￥", 180, 128);
				ctx.setFillStyle('#E80013');
				ctx.setFontSize(20);
				ctx.fillText(selectGoods.nowSale, 195, 128);
				//原价
				ctx.setFillStyle('#333');
				ctx.setFontSize(15);
				ctx.fillText('原价:￥' +selectGoods.sale , 180, 150);
				ctx.rect(180, 145, 88, 1);
				ctx.setFillStyle('#333');
				ctx.fill();
				//底部
				ctx.setFillStyle('#fff');
				ctx.setFontSize(15);
				ctx.fillText("只剩最后一小时", 50, 217);
				//开始画画完后生成新的临时图片地址
				ctx.draw(false, () => {
					setTimeout(()=>{
						uni.canvasToTempFilePath({
							canvasId: 'shareFriend',
							success: (res) => {
								uni.hideLoading();
								this.picTempUrl=res.tempFilePath;
							}
						});
					}, 200);
				})
			},
			//获取手机相册权限
			getPhotosAlbumAuth(){
				uni.getSetting({
					success:(res)=> {
						if (!res.authSetting['scope.writePhotosAlbum']) {
							uni.authorize({
								scope: 'scope.writePhotosAlbum',
								success:()=> {
									this.saveImage();
								}
							})
						} else {
							this.saveImage();
						}
					}
				})
			},
			//保存海报
			saveImage(){
				uni.saveImageToPhotosAlbum({
					filePath: this.picTempUrl,
					success: (data)=> {
						uni.showToast({
							title: "图片保存成功",
							icon: "success",
							mask: true
						})
					},
					complete: () => {
						this.posterStatus=false;
					}
				})
			},
		}
	}
</script>

<style lang="scss">
	.listWrap{
		height: calc(100vh - 140upx);padding: 20upx 0;font-size: 32upx;color: #666;
		.goodsItem{
			display: flex;flex-wrap: nowrap;margin: 10upx 20upx;box-shadow: 0 0upx 20upx -5upx #999;
			.imgBox{
				position: relative;width: 300upx;max-height: 300upx;overflow: hidden;
				image{width: 100%;max-height: 100%;}
				.chooseFlag{position: absolute;top: 10upx;left: -70upx;width:200upx;text-align: center;transform: rotate(-45deg);color: #fff;background: #007AFF;}
			}
			.goodsInfo{
				width: 45%;padding: 20upx;line-height: 80upx;
				.goddsName{font-size: 40upx;font-weight: bold;}
				.sale{color: #999;text-decoration: line-through;}
				.nowSale{
					.heightLight{color:#ff7800;}
					.textBig{font-size: 40upx;}
				}
			}
			.chooseBtn{
				width: 25%;line-height: 300upx;font-size: 40upx;text-align: center;color: #fff;background: #10bb20;
				&:active{
					background: #009900;
				}
			}
		}
	}
	.shareBtnList{
		height: 60upx;padding: 20upx;border-top:1px solid #eee;text-align: center;
	}
	.sharePlaneWrap{
		position: fixed;top: 0;left: 0;width: 100vw;height: 100vh;background: rgba(0,0,0,.5);
		.sharePlane{
			width: 600upx;height: 450upx;padding: 40upx;margin: 0 auto;margin-top: calc(50vh - 200upx);border-radius: 50upx;background: #F8F8F8;
			.top{
				display: flex;flex-wrap: nowrap;
				&>button{
					width: 50%;padding: 20upx;border: none !important;text-align: center;
					image{width: 60%;}
					&:active{
						background: #eee;
					}
					&::after{border: none;}
				}
			}
		}
	}
	.posterPreview{
		position: fixed;top: 0;left: 0;width: 100vw;height: 100vh;padding-top: 100upx;z-index: 9;text-align: center;background: rgba(0,0,0,.8);
		image{width: 80%;}
		button{width: 80%;position: absolute;bottom: 55px;left: 10%;}
	}
	.canvas-box{width:0px;height:0px;overflow:hidden;position: absolute;top:-1000px;}
</style>
