<template>
	<view class="container">
		<uni-row class="demo-uni-row">
			<uni-col :xs="6" :sm="6" :md="6" :lg="6" :xl="6" >
				<view class="demo-uni-col dark">

							<uni-title type="h1" title="历史记录" color="#027fff"></uni-title>
							<uni-list :border=true>
										<uni-list-item  :title="(item.ws_url)" :show-badge="true" :clickable="true" :badge-text="(item.count)"  @click="setWsUrl(item.ws_url)" v-for="(item,index) in history" :key="index" ></uni-list-item>									
							</uni-list>		

				</view>
				
			</uni-col>
			<uni-col :xs="1" :sm="1" :md="1" :lg="1" :xl="1">
			</uni-col>
			<uni-col :xs="17" :sm="17" :md="17" :lg="17" :xl="17">
						<view>
							<view class="demo-uni-col light">
								<uni-title type="h1" title="测试操作区" color="#027fff"></uni-title>
							</view>
							<uni-row class="demo-uni-row">
								<uni-col :xs="12" :sm="12" :md="12" :lg="12" :xl="12" >
									<uni-easyinput prefixIcon="paperclip"  type="text" v-model="connect.ws_url" placeholder="请输入ws地址" />
					
								</uni-col>
								<uni-col :xs="4" :sm="4" :md="4" :lg="4" :xl="4">
									<uni-easyinput type="number"  v-model="connect.count" placeholder="并发量" />				
								</uni-col>
								<uni-col :xs="4" :sm="4" :md="4" :lg="4" :xl="4">
										<button class="conn-btn" @click="connectFunc" >连接</button>
								</uni-col>
								<uni-col :xs="4" :sm="4" :md="4" :lg="4" :xl="4"  style="padding-left: 20px;padding-top: 3px;">
										<uni-tag :text="''+(successConn)" type="success" :circle="true"></uni-tag>
								</uni-col>
								
							</uni-row>
						</view>
	
						<view style="padding-top:40rpx ;">
								<uni-row class="demo-uni-row" >
									
									<uni-col :xs="11" :sm="11" :md="11" :lg="11" :xl="11">
										<view class="demo-uni-col light">
											<uni-title type="h1" title="发送区域(不支持并发调试)" color="#027fff"></uni-title>
										</view>
										<view>
											
											<uni-row class="demo-uni-row ping-box" >

												<uni-col :xs="4" :sm="4" :md="4" :lg="4" :xl="4" class="ping-border" >
												每隔
												</uni-col>
												<uni-col :xs="4" :sm="4" :md="4" :lg="4" :xl="4" class="ping-border">
													 <input type="number"  v-model="interval" placeholder="1" style="text-align: center;" />
												</uni-col>
												<uni-col :xs="6" :sm="6" :md="6" :lg="6" :xl="6" class="ping-border">
												秒发送心跳
												</uni-col>
												<uni-col :xs="4" :sm="4" :md="4" :lg="4" :xl="4" class="ping-border">
												Ping
												</uni-col>
												<uni-col :xs="6" :sm="6" :md="6" :lg="6" :xl="6" class="ping-border">
													<button class="ping-btn" :class="[pingSwitch?'cancelPing':'sendPing']" @click="startPing" >{{pingBtnText}}</button>
												</uni-col>
											</uni-row>
		
										</view>		
										<view style="padding-top: 10rpx;">
											<uni-easyinput type="textarea" autoHeight style="background-color:#ced4da; " v-model="content" placeholder="发送到服务器的数据包内容"></uni-easyinput>
										</view>
										<view style="padding-top: 10rpx;">
											       <button class="button" type="primary" style="background-color: #28a745;" @click="sendMsg">发送</button>
										</view>
										
										<view class="demo-uni-col light" style="padding-top: 20rpx;">
											<uni-title type="h1" title="发送记录" color="#027fff"></uni-title>
										</view>
										<view class="demo-uni-col light" style="padding-top: 20rpx;">
											<hr>
										</view>
										<view class="demo-uni-col light send-box" >
											<view v-for="val,index in sendbox" :key="index">
												<span style="color: #28a745;">{{val.time_}}	 => </span> <pre> {{val.msg}} </pre>
											</view>
	
										</view>
									</uni-col>
									<uni-col :xs="2" :sm="2" :md="2" :lg="2" :xl="2" style="text-align: center;">
										<view class="line"></view>
									</uni-col>
									<uni-col :xs="11" :sm="11" :md="11" :lg="11" :xl="11">
											<view class="demo-uni-col light">
												<uni-title type="h1" title="消息接收区" color="#027fff"></uni-title>
											</view>
											<view class="demo-uni-col light">
												   
												<checkbox-group  @change="checkboxChange"  >
													<label  v-for="item in checkBoxItems" :key="item.value">
														<checkbox :value="item.value" :checked="item.checked" />{{item.name}}
													</label>

												</checkbox-group>
											</view>
											<view class="demo-uni-col light msg-box" >
												<view v-for="val,index in msgbox" :key="index">
													<span style="color: #28a745;">{{val.time_}} => </span> 
													
													<pre>
														{{val.msg}}	
													</pre>
												</view>

											</view>
									</uni-col>
								</uni-row>
						</view>
			</uni-col>
		</uni-row>
		<uni-popup ref="popup" type="message">
			<uni-popup-message type="error" :message="errorMsg" :duration="2000"></uni-popup-message>
		</uni-popup>
	</view>
</template>
<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
<script>
	var socketOpen = false;
	var socketMsgQueue = [];
	Date.prototype.Format = function (fmt) { // author: meizz
	  var o = {
	    "M+": this.getMonth() + 1, // 月份
	    "d+": this.getDate(), // 日
	    "h+": this.getHours(), // 小时
	    "m+": this.getMinutes(), // 分
	    "s+": this.getSeconds(), // 秒
	    "q+": Math.floor((this.getMonth() + 3) / 3), // 季度
	    "S": this.getMilliseconds() // 毫秒
	  };
	  if (/(y+)/.test(fmt))
	    fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
	  for (var k in o)
	    if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
	      return fmt;
	}
	export default {
		
		
		data() {
			return {
				href: 'https://uniapp.dcloud.io/component/README?id=uniui',
				connect:{
					ws_url:"ws://127.0.0.1:8083/ws",
					count:10,
				},
				value:"",
				interval:1,
				pingSwitch:false,
				intervalId:0,
				pingBtnText:"执行",
				msgbox:[],
				sendbox:[],
				content:"",
				successConn:"0",
				cacheKey:"ws_url_list",
				history:[],
				errorMsg:"",
				jsonCheckBox:true,
				checkBoxItems: [{
				                        value: '1',
				                        name: 'json解码'
				                    },
				                    {
				                        value: '2',
				                        name: '暂停显示新消息',
				      
				                    },
				                    {
				                        value: '3',
				                        name: '只显示新消息'
				                    },
				],
				WebSocketPool:[],
				
			}
		},
		created() {
			this.getCache()
		},
		methods: {
			sendMsg(){
				if(this.content == ""){
					this.errorMsg = "请输入内容";
					this.$refs.popup.open()
					return ;
				}
				if(this.WebSocketPool.length <= 0){
					this.errorMsg = "请先连接";
					this.$refs.popup.open()
					return ;
				}
		
				var time = new Date().Format("yyyy-MM-dd hh:mm:ss");
				this.sendbox.push({msg:this.content,time_:time})
				this.WebSocketPool[0].send({data: this.content});
			
			},
			connectFunc(){
					let this_ = this;
					this_.successConn = 0;
					
						for(let i=1;i<= this_.connect.count;i++){
							try{
								var socketTask = uni.connectSocket({
									url: this.connect.ws_url,
									header:{
										'origin':'127.0.0.1'
									},
									complete: ()=> {}
								});
								socketTask.onOpen(function(){
									   this_.successConn += 1;
									   this_.setCache();
								})
								
								socketTask.onMessage(function(res){
									console.log(res.data)
									if(!this_.checkBoxItems[1].checked){
										
										if(this_.checkBoxItems[2].checked){
											this_.msgbox = [];
										}
										var data = this_.parseJSON(res.data);
										var time = new Date().Format("yyyy-MM-dd hh:mm:ss");
										this_.msgbox.push({msg:data,time_:time});
									}

								})
								
								socketTask.onClose(function(res){
									  console.log('WebSocket 已关闭！');
								})
								
								this.WebSocketPool.push(socketTask);
							
							}catch(e){
								console.log('第'+i+'次连接失败',e)
							}
							
						}
						
			},
			setCache(){
				
				let data = [];
				try {
				    const value = uni.getStorageSync(this.cacheKey);
				    if (value) {
				        let isAdd = 1;
						value.forEach((elem, index) => {
							if(elem.ws_url == this.connect.ws_url){
								isAdd = 0;
								elem.count = Number(elem.count);
								elem.count += 1;
								elem.count = elem.count.toString();
							}
						//  	console.log(elem, elem.ws_url , this.connect.ws_url);
						});
					
						data = value;
						if(isAdd){
							data.push({count:"1",ws_url:this.connect.ws_url})
						}
				    }else{
						data.push({count:"1",ws_url:this.connect.ws_url})
					
					}
				   uni.setStorageSync(this.cacheKey, data);
					this.getCache();
				} catch (e) {
				    // error
						console.log('get:err',e)
				}
		
		
			},
			getCache(){
				 try {
				     const value = uni.getStorageSync(this.cacheKey);
				     if (value) {
				         this.history = value;
				     }
				 } catch (e) {
				 }
			},
			setWsUrl(ws_url){
				this.connect.ws_url = ws_url;
			},
			 checkboxChange: function (e) {
				var items = this.checkBoxItems,
					values = e.detail.value;
				for (var i = 0, lenI = items.length; i < lenI; ++i) {
					const item = items[i]
					if(values.includes(item.value)){
						this.$set(item,'checked',true)
					}else{
						this.$set(item,'checked',false)
					}
				}
			},
			parseJSON(str) {
			    if (typeof str == 'string') {
			        try {
			            var obj=JSON.parse(str);
			            if(typeof obj == 'object' ){
							if(this.checkBoxItems[0].checked){
								 return JSON.stringify(obj, null, 4);
							}else{
								return str;
										
							}
			   
			            }
			
			        } catch(e) {
			            
						console.log('err',e)
			        }
			    }
				return str;
			
			},
			sendPing(){
				
				this.WebSocketPool[0].send({data: parseInt(Math.random()*1000+1,10) });
				console.log("开始定时"+parseInt(Math.random()*1000+1,10));
			},
			startPing(){
				
				if(this.WebSocketPool.length <= 0){
					this.errorMsg = "请先连接";
					this.$refs.popup.open()
					return ;
				}
				this.pingSwitch = !this.pingSwitch;
				if(this.pingSwitch){
					this.pingBtnText = "取消";
					this.intervalId = setInterval(this.sendPing,this.interval * 1000)
				}else{
					if(this.intervalId){
						clearInterval(this.intervalId)
						this.intervalId = 0;
					}
					this.pingBtnText = "执行";
					console.log("取消定时");
				}
				
			}
		},

	}
</script>

<style>
	.container {
		padding: 20px;
		font-size: 14px;
		line-height: 24px;
		min-width: 1200px;
	}
	.msg-box{
		padding-top: 20rpx;
		height: 630px; 
		overflow-y:auto;
		 overflow-x:auto;
	}
	.conn-btn{
		background-color: #28a745;
		height: 38px;
		color:white;
		line-height: 38px;
	}
	.ping-box{
		background-color: #ced4da;
		border-top: #808080 1px solid;
		border-bottom: #808080 1px solid;
	}
	.send-box{
		padding-top: 20rpx;
		height: 300px; 
		overflow-y:auto; 
		overflow-x:auto;
	}
	.title{
		background-color: #007AFF;
		height: 53px;
	}
	.chat-custom-text {
	    font-size: 12px;
	    color: red;
	}
	.ping-border{
		
		border-left: #808080 1px solid;text-align: center;
	}
	.line{
	 	background:#E7E7E7;/*背景色为浅灰色*/
		width:0.6px;/*设置宽高*/
		height:730px;
		position:relative;/*调整位置*/
		left:30px;
		float:left;/*让此div与前面的并排显示*/
	}
	.sendPing{
		
		background-color: #28a745;
	}
	.cancelPing{
		background-color: #666666;
	}
	.ping-btn{
/* 		background-color: #28a745; */
		height: 28px;
		color:white;
		line-height: 28px;
		border-radius: 0;
	}

</style>
