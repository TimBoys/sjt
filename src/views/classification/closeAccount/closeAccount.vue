<template>
	<div class="app-init ca-cont">
    <div class="classification-header">
    	<header-back :title="headTitle"></header-back>

    </div>	
	<div class="absolute closeAccountCont">
		<!--头部三个选择s-->
			<div class="groupCont">
			    <!--<tab bar-active-color="#FDA544" >
			      <tab-item selected active-class="tabItem">{{$t('closeAccount.ToStoreComeUndone')}}</tab-item>
			      <tab-item disabled active-class="tabItem" @click.native="nextWait">{{$t('closeAccount.DeliveryDistribution')}}</tab-item>
			    </tab>-->
				<group label-width="4.5em" label-margin-right="2em" gutter="0" label-align="left" class="groupItem" >
						<!--<popup-picker title="配送方式" :data="list" v-model="value5" value-text-align="left" ></popup-picker>-->
						<cell :title="modeDistribution" value-align="left" primary="content" >
					      <checker v-model="checkerDemo" radio-required default-item-class="demo1-item" selected-item-class="demo1-item-selected" disabled-item-class="demo1-item-disabled" @on-change="changeChecker">
					        <checker-item :value="item" v-for="(item, index) in checkerItems" :key="index">{{item.value}}</checker-item>
					        <checker-item :value="demo3" disabled @click.native="nextWait">{{takeOut}}</checker-item>
					      </checker>					
						</cell>
						<cell :title="mustAddress" value-align="left" primary="content"   :value="storeAddress">
							
						</cell>
						<cell :title="invitethephone" value-align="left" primary="content"  :value="storePhone">
							 <!--@click.native="showPhone = true"-->
						</cell>
						<popup-picker :title="appointmentTime" v-show="isShowOrderTime" v-model="formatDemoValue" value-text-align="left" :data="ppAllYuyueTime" :display-format="format"></popup-picker>
						<popup-picker :title="modeOfPayment" :data="listSel" v-model="valueDefault" value-text-align="left" ></popup-picker>
						
				</group>	

			</div>
		<!--头部三个选择e-->
		
		<!--商品详情-->
		<div class="goodsDetail">
			<div class="gd-title">
				{{gdTitle}}
			</div>
			<div class="gd-cont" v-if="showInitData">
				<div class="gdc-detail" v-for="(item,index) in allGoods.allGDPage">
					<x-img  class="gdcd-img" v-lazy="item.goodsItem.goodsPictureRound"></x-img>
					<div  class="gdcd-price">
						<div class="gdcdp-right">
							<div class="gdcdpr-title">{{item.goodsItem.goodsName}}</div>
							<div>{{item.iGAGAllGuige}} <span class="ft2" v-if="item.goodsItem.goodsGuigePrice">(C${{item.goodsItem.goodsGuigePrice}})</span></div>
							<div>x{{item.itemOneGuigeLen}}</div>
						</div>
						<div class="gdcdp-plus">
							<span>C${{item.hasGuigePrice * item.itemOneGuigeLen}}</span>
							<!--<span>${{item.goodsItem.goodsPrice}}</span>
							<span>${{item.goodsItem.goodsGuigePrice}}</span>-->
						</div>
					</div>
				</div>
				<div class="gdc-footer">
					<div class="gdcf-allPrice">
						{{$t('closeAccount.originalPrice')}}：
						<span class="red">C${{allGoods.allGDOrigPrice}}</span>
					</div>
					<div class="gdcf-allPrice">
						{{$t('closeAccount.discounts')}}<span class="ft2">({{promotionName}})</span>：
						<span class="red">C${{allGoods.allGDDiscount}}</span>
					</div>
					
					<div class="gdcf-allPrice">
						{{$t('closeAccount.subtotal')}}：
						<span class="red">C${{allGoods.allGDOrderPrice}}</span>
					</div>
					<group :title="theSellerMessage" class="gdc-textarea">
  						<x-textarea  name="detail"  :max="maxNum" :placeholder="holdSay" :show-counter="true" v-model="textAreaValue"></x-textarea>
					</group>
				</div>
			</div>
		</div>
		<!--商品详情-->
		
		
	</div>		
		<!--底部结算按钮s-->
	<div class="classification-footer absolute">
		<div class="cf-left">
			<div class="cfl-cont" >
				<x-img v-lazy="sjtLogo" class="iconfont icon-gouwuche ftl-gwc"></x-img>
				<div class="ftl-redPoint" v-if="allGoods.allGDLength">
					{{allGoods.allGDLength}}
				</div>
			</div>
		</div>
				<div class="cf-center">
			{{$t('classification.totalPrice')}}:C${{allGoods.allGDOrderPrice}}
		</div>
				<div class="cf-right" @click="updateAccount()">
			{{$t('closeAccount.submitOrder')}}
		</div>
				<!--<a href="../../../static/alipay.html" target="_blank" class="cf-right" @click="updateAccount()">
			{{$t('closeAccount.submitOrder')}}
		</a>		-->
	</div>
	<!--底部结算按钮e-->
		
		
	<div >
      <confirm v-model="showPhone"
      show-input
      ref="confirm5"
      :title="showPhoneTitle"
       @on-cancel="onCancel"
      @on-confirm="onConfirm5"
      @on-hide="onHide">
      </confirm>
    </div>
    
    <!--loadding弹出框-->
	<loading :show="showLoading" is-show-mask :text="showText"></loading>

	 <toast  v-model="showPositionValue"  type="text" :time="2000" is-show-mask :text="enjoyLooking" :position="position"></toast>
	
	<!--二维码支付-->
    <div class="dialogGoodsDet">
	      <x-dialog v-model="showHideOnBlur" @on-show="pcPayShow" @on-hide="pcPayHide" class="dialog-demo" hide-on-blur :dialog-style="{'width':'60%','max-width':'6rem','overflow':'visible'}">
	        <div class="img-box">
	          <div class="dgdCont">
	          	{{iotPayMsg}}
	          </div>
	          <img :src="pcPayUrl" style="max-width:6rem">
	        </div>
	      </x-dialog>
    </div>		
    
	</div>
	
</template>

<script>
import headerBack from "../../../components/header-back";
import { Confirm,XImg,Divider,PopupPicker,Tab,TabItem,XTextarea } from "vux";
import VueDB from "../../../util/vue-db/vue-db-long.js";
import shopCarTool from "../../../util/shop-car-tool/index.js";
import isMobile from "../../../util/isMobile/isMobile.js";
import formatTime from "../../../util/formatTime/formatTime.js";
import _ from 'lodash'
//微信支付
var onBridgeData = {};
var DB = new VueDB();
var handler = null;
var num = 1; //IOTpay支付订单区别
var loopPayInter = null; //定时查询订单

export default{
	name:"closeAccount",
	data(){
		return{
			showHideOnBlur: false,
			showPositionValue: false, //toast弹出
			position: 'default',
			headTitle:this.$t('closeAccount.submitOrder'), //加入购物车
			mustAddress:this.$t('closeAccount.MustAddress'), //自取地址
			invitethephone:this.$t('closeAccount.Invitethephone'), //自取电话
			appointmentTime:this.$t('closeAccount.appointmentTime'), //预约时间
			modeOfPayment:this.$t('closeAccount.modeOfPayment'), //支付方式
			theSellerMessage:this.$t('closeAccount.theSellerMessage'), //给卖家留言
			holdSay:this.$t('closeAccount.holdSay'), //写下想对卖家说的话
			enjoyLooking:this.$t('closeAccount.enjoyLooking'), //敬请期待
			checkerItems: [{
		        key: '0',
		        value: this.$t('closeAccount.reserve')
		      }, {
		        key: '1',
		        value:  this.$t('closeAccount.eatIn')
		      }],
		    checkerDemo: {key: '1', value:  this.$t('closeAccount.eatIn')},
   			demo1: {key: '1', value:  this.$t('closeAccount.eatIn')},
   			demo2: {key: '0', value: this.$t('closeAccount.reserve')},
   			demo3: {key: '2', value: this.$t('closeAccount.takeOut')},
   			takeOut:this.$t('closeAccount.takeOut'),
   			modeDistribution:this.$t('closeAccount.modeDistribution'),
			gdTitle:"",
			sjtLogo:"../../../static/images/mine/sjtLogo1.jpg",
			sjtStripeLogo:"../../../static/images/mine/circleLogo.png",
			listPc: [['VISA/Master/Debit','微信支付','支付宝']],  //pc浏览器
			listMo: [['VISA/Master/Debit','微信支付']], //,'微信支付','支付宝' //手机浏览器
			listWx: [['VISA/Master/Debit','微信支付']], //微商城
			isPcMoWx:'',  //判断不同设备
			listSel: [['VISA/Master/Debit']], //根据设备选择支付方式
			valueDefault: ['VISA/Master/Debit'],  //默认支付方式
			pcPayUrl:"", //pc支付二维码地址
			iotPayMsg:"", //提示pc支付
			showPhone:false,
			showPhoneTitle:this.$t("closeAccount.changePhone"),
			formatDemoValue: [],				//默认预约时间
			ppAllYuyueTime:[], //所有预约时间
			isShowOrderTime:false,
			format: function (value, name) {
				console.log(value)
				if (value[1] == "closeDoor") {
					return `${value[0]}`
				}else{
		        return `${value[0]}:${value[1]}`
		     }
			},
		    allGoods:[],
		    textAreaValue:"",
		    promotionId:null,
		    promotionName:null,
		    storeAddress:"shanghai",
		    storePhone:"110",
		    showLoading:false, //loading加载
		    showInitData:false, //初始化数据加载
		    showText:this.$t("reminder.dataLoading"),
		    orderNo:null, //订单编号
		    goodsId:null, //商品id
		    maxNum:250,  //最长留言
		}
	},
	components:{
		Confirm,
		headerBack,
		XImg,
		PopupPicker,
		Tab,
		TabItem,
		XTextarea
	},
	mounted:function(){
		this.shopCar = new shopCarTool(this.$store);
		this.isMobile = new isMobile();
		
		//商品个数为0退出到所有商品
//		console.log("商品个数为01退出到所有商品")
//		console.log(this.shopCar.length())
		if (!this.shopCar.length()) {
			console.log("isNUll")
			this.$vux.toast.show({
				text: this.$t("closeAccount.pleaseSelGoods"),
				type: "text",
			})
			setTimeout(()=>{
				this.$router.openPage("/classification");
			},1000)
			return false;
		}	
		
	    this.showLoading = true;	
	    setTimeout(()=>{
	    	this.showLoading = false;
	    },20000)		
	    
		//初始化预约时间
//		this.initYuyueTime();

		//根据不同设备初始化支付方式
		this.initPayStyle();
		
		this.queryPromotionByStoreNoNation(); //查询活动id
		
		this.initPayData(); //初始化stripe支付
		
	},
	methods:{
		//展示规格end
	    open(link){
	    	this.$router.openPage(link);
	    },
	    initPayStyle(){
			var telUserNo = DB.getItem("telUserNo").toJson();
	    	//wxcode存在是微商城
//	    	if (DB.getItem("wxCode").toString() || telUserNo.weixinOpenid) {
	    	if (DB.getItem("wxCode").toString()) {
	    		this.isPcMoWx = "wx";
	    		this.listSel = this.listWx;
	    	}else{
	    		//移动端 / pc端
	    		if(this.isMobile.isMobile()){
	    			this.isPcMoWx = "mo";
	    			this.listSel = this.listMo;
	    		
	    		}else{
	    			this.isPcMoWx = "pc";
	    			this.listSel = this.listPc;
	    		}
	    		
	    	}
	    	
	    },
	    //stripe支付
	    initPayData(){
     			 var _this = this;
     		     handler = StripeCheckout.configure({
//					 key:'pk_test_ujKHOw9xZM2QYfJwDZIt890W',  //测试key
       		         key: 'pk_live_3aqw1J17VC1gcSxl29khgL3u',  //TODO:正式key,正式发布时替换
     		        image: _this.sjtStripeLogo,
     		        locale: 'auto',
     		        allowRememberMe:true,
     		        token: function(token) {
     		            // You can access the token ID with `token.id`.
     		            // Get the token ID to your server-side code for use.
     		            console.log(token)
     		            //TODO:发起http请求，将token、amount、description等参数发送给服务器
     		            // 接口地址/stripe/charge
						var data={
     		                token:token.id,
							amount:_this.allGoods.allGDOrderPrice * 100,
							description:_this.$t("closeAccount.milkyTea"),
                            orderNum:_this.orderNo   //订单编号
						}
						_this.showLoading = true;	
						setTimeout(()=>{
					    	_this.showLoading = false;
					    },20000)
     		           _this.$http.post("/stripe/charge",data).then((res) => {
     		              console.log(res)
	     		            if(DB.getItem("localLang").toString() == "en"){
								var ErrorMsg = res.data.usErrorMsg;
							}else{
								var ErrorMsg = res.data.cnErrorMsg;
							}
     		               if(res.status == 200) {
	     		               	_this.showLoading = false;	
     		               		if(res.data.rspCode == "00000"){
//   		               		_this.finishPayModfiyOrder();
     		               		//清除店铺缓存
								_this.shopCar.removeAll();
								_this.$vux.toast.show({
									text: _this.$t("closeAccount.paymentSuccess"),
									type: "text",
								}) 
								
								_this.$router.openPage("/mineOrder");
     		               }else{
     		               		_this.$vux.toast.show({
									text: ErrorMsg,
									type: "text",
								})
     		               }
     		               }
     		               
     					}).catch((res) => {
     						console.log(res)
     						_this.showLoading = false;	
     					})   		        
     		        }
     		    });     			
	    	
	    	    window.addEventListener('popstate', function() {
	    	    	handler.close();
			    });
			        	
	    },
	   stripePay(){
	    	var _this = this;
 			 handler.open({
 	            name: '素匠·泰茶',
   	            description: _this.$t("closeAccount.commodityInformation"),
// 	            currency: 'usd',//美元
   	            currency: 'cad',//加币
 	            amount: _this.allGoods.allGDOrderPrice * 100   //TODO:金额，单位分，变量,需乘以100
 	        });		    	
	    },
	    
	    //pc支付宝支付
	    pcAliWxPayData(channelId,extra){
	    	console.log(channelId)
	    	console.log(extra)
	    	var _this = this;
	    	
	    	console.log("pcAliWxPayData")
	    	var commodityInformation = _this.$t("closeAccount.commodityInformation");
	    	var data = {
	    		mchOrderNo:_this.orderNo+"@"+num++,
				channelId:channelId,
	    		currency:"CAD",
	    		amount:_this.allGoods.allGDOrderPrice * 100,
//	    		device:"WEB",
	    		subject:"素匠·泰茶",
	    		body:commodityInformation,
				extra:extra,
	    	}
	    	console.log(data)
	    	
			_this.$http.post("/iotpay/charge",data).then((res) => {
				console.log(res)
	    		console.log("charge")
				if(res.status == 200 && res.data.rspCode == "00000") {
				console.log(JSON.parse(res.data.data))
				var resData = JSON.parse(res.data.data);
				console.log(resData.codeUrl)
//				location.href = JSON.parse(res.data.data).payUrl;
					if(channelId == "ALIPAY_QR"){
						this.pcPayUrl = resData.payUrl;
						this.iotPayMsg = "请打开手机支付宝扫一扫！";
					}else if(channelId == "WX_NATIVE"){
						this.pcPayUrl = "http://qr.topscan.com/api.php?text=" + resData.codeUrl;
						this.iotPayMsg = "请打开手机微信扫一扫！";
					}
					this.showHideOnBlur = true;
					
				}else{
					_this.$vux.toast.show({
						text: res.data.data.retMsg,
						type: "text",
					})
				}
			}).catch((err) => {
           		
			})		    	
	    	
	    	
	    },
	    //微商城内微信支付
	    wPayData(channelId,extra){
	    	console.log("wPayData")
	    	console.log(channelId)
	    	console.log(extra)
	    	var _this = this;
	    	
	    	console.log("wPayData")
	    	var commodityInformation = _this.$t("closeAccount.commodityInformation");
	    	var data = {
	    		mchOrderNo:_this.orderNo+"@"+num++,
				channelId:channelId,
	    		currency:"CAD",
	    		amount:_this.allGoods.allGDOrderPrice * 100,
//	    		device:"WEB",
	    		subject:"素匠·泰茶",
	    		body:commodityInformation,
				extra:extra,
				returnUrl:"https://www.primes-thaitea.com/wxindex.html#/home"
	    	}
	    	console.log(data)	    	

			_this.$http.post("/iotpay/charge",data).then((res) => {
				console.log(res)
	    		console.log("charge2222")
				if(res.status == 200 && res.data.rspCode == "00000") {
					console.log(JSON.parse(res.data.data))
					var resData = JSON.parse(res.data.data);
					if(resData.success){
						window.location.href = resData.url;
					}
				}else{
					_this.$vux.toast.show({
						text: res.data.data.retMsg,
						type: "text",
					})
				}
			}).catch((err) => {
           		
			})
	    	
	    },
	    
	    //开始pc轮询支付成功
	    pcPayShow(){
	    	console.log("pcPaySHow")
	    	loopPayInter = setInterval(()=>{
	    		this.loopPcPaySuc();
	    	},10000)
	    },
	    //关闭pc轮询支付成功
	    pcPayHide(){
	    	console.log("pcPayHide")
	    	clearInterval(loopPayInter);
	    	this.loopPcPaySuc();
	    	
	    },
	    loopPcPaySuc(){
	    	var _this = this;
	    	_this.$http.get("/userOrderInfo/selectbyorderno",{params:{
	    		orderNo:_this.orderNo
	    	}}).then((res) => {
//	    		console.log("selectbyorderno")
				console.log(res)
	            if(DB.getItem("localLang").toString() == "en"){
					var ErrorMsg = res.data.usErrorMsg;
				}else{
					var ErrorMsg = res.data.cnErrorMsg;
				}	
				if(res.status == 200){
				if(res.data.rspCode == "00000" && res.data.data.payStatus == 1) {
					_this.shopCar.removeAll();
					_this.$vux.toast.show({
						text: _this.$t("closeAccount.paymentSuccess"),
						type: "text",
					}) 
					
					_this.$router.openPage("/mineOrder");					
				}else{
//					_this.$vux.toast.show({
//						text: ErrorMsg,
//						type: "text",
//					})
				}
				}
				
			}).catch((err) => {
           		
			})	
	    },
	    
	    //初始化预约时间
	    initYuyueTime(start,end){
	    	//是否为预约单子
			console.log("DB.getItem(isOrder).toString()")
//			console.log(start.slice(0,2))
//			console.log(end.slice(0,2))
			var startTime = start.slice(0,2)
			var endTime = end.slice(0,2)
			console.log(DB.getItem("isOrder").toString())
			if (DB.getItem("isOrder").toString() == "isOrder") {
				this.isShowOrderTime = true;
				this.checkerDemo = this.demo2;
			}else{
				this.isShowOrderTime = false;
				this.checkerDemo = this.demo1;
			}
			
	    	var theTimeHour = [];
	    	var theTimeMin = [];
//	    	var n=new Date().getTime();
//	    	var theSecondStart = new Date(n + 1000*60*30).getMinutes() < 10 ? "0" + new Date(n + 1000*60*30).getMinutes() : new Date(n + 1000*60*30).getMinutes();
	    	console.log("theSecondStart")
//	    	console.log(theSecondStart)
	    	for (var i = startTime ; i < endTime; i++) {
	    		var iHour = i < 10 ? "0"+i : i;
	    		theTimeHour.push(iHour);
	    	}
	    	for (var j = 0 ; j <60 ; j++) {
	    		var iMin = j < 10 ? "0"+j : j;
	    		theTimeMin.push(iMin);
	    	}
//	    	console.log(theTimeMin)
	    	var nowHour = new Date().getHours() < 10 ? "0"+new Date().getHours() : new Date().getHours();
	    	var n=new Date().getTime()  - 1000*60*30;
	    	var nowMinutes = new Date(n).getMinutes() < 10 ? "0"+new Date(n).getMinutes() : new Date(n).getMinutes();
//	    	console.log(nowHour)
//	    	console.log(theTimeHour)
//	    	console.log(theTimeHour.indexOf(nowHour))
			if (theTimeHour.indexOf(nowHour) == -1) {
				this.formatDemoValue.push(this.$t("closeAccount.shopHasNot"),"closeDoor")
			}else{
				var theNowHour = nowHour == 17 ? theTimeHour.slice(theTimeHour.indexOf(nowHour)) : theTimeHour.slice(theTimeHour.indexOf(nowHour) + 1)
				console.log(theNowHour)
//				console.log(theTimeMin.indexOf(nowMinutes))
//				console.log(theTimeMin.slice(theTimeMin.indexOf(nowMinutes),59))
//				this.ppAllYuyueTime.push(theNowHour,theTimeMin.slice(theTimeMin.indexOf(nowMinutes),59));  //半个小时以后
				this.ppAllYuyueTime.push(theNowHour,theTimeMin);
				this.formatDemoValue.push(theNowHour[0],theTimeMin.slice(theTimeMin.indexOf(nowMinutes),59)[0])
			}
	   },
		
		//模态框s
	    onHide(){
	    	console.log("关闭模态框")
	    },
	    onCancel(){
	    	console.log("取消模态框")
	    },
	    onConfirm5 (value) {
	      console.log(value)
	      this.storePhone = value;
	    },
	    //获取店铺的活动
	    //8/15现在是默认第一个活动（第二件半价）
	    queryPromotionByStoreNoNation(){
	    	DB.getItem("storeList").toJson().forEach((item,index)=>{
	    		if (DB.getItem("storeNo").toString() == item.storeNo) {
	    			console.log("DB.getItem(storeNo).toString()")
					console.log(item)
					//初始化预约时间
					this.initYuyueTime(item.reserveBeginTime,item.reserveEndTime);					
					
	    			this.storeAddress = item.storeAddress;
	    			this.gdTitle = "素匠·泰茶("+item.storeAddress+")";
	    			this.storePhone = item.storePhone;
	    		}
	    	})
	    	
			this.$http.get("/userLogin/queryPromotionByStoreNoNation",{
				params:{
					storeNo:DB.getItem("storeNo").toString(),
					lang:DB.getItem("localLang").toString()
				}
			}).then((res) => {
				console.log(res)
				if(res.status == 200 && res.data.rspCode == "00000" && res.data.data.data.length) {
					this.promotionId = res.data.data.data[0].promotionId;
					this.promotionName = res.data.data.data[0].promotionName;
					this.initUserOrder();  //初始化订单
				}else{
	               		this.$vux.toast.show({
							text: "该店铺活动已结束！",
							type: "text",
						})							
					this.$router.openPage("/classification");
				}
			}).catch((err) => {
				console.log(err)
			})		    	
	    	
	    },
	    //初始化订单
	    initUserOrder(){
	    	console.log("queryPromotionByStoreNoNation-telUserNo")
				var telUserNo = DB.getItem("telUserNo").toJson();
				var getAllShopCar = this.shopCar.getAll();
				var listTeaOrderDetails = [];
//				console.log(getAllShopCar)
//				console.log(DB.getItem("allGoodsAttrs").toJson())
				for (var itemKey in getAllShopCar) {
					for (var i =0; i < getAllShopCar[itemKey].itemGuige.length; i++) {
//						console.log(getAllShopCar[itemKey].itemGuige[i].initGuiGeSC)
						for (var k = 0; k < getAllShopCar[itemKey].itemGuige[i].itemOneGuigeLen; k++) {
							var pushGoods = {};
							var oneStyleItemArr = [];
							pushGoods.goodsId = getAllShopCar[itemKey].goodsId;
							oneStyleItemArr = JSON.parse(getAllShopCar[itemKey].itemGuige[i].initGuiGeSC);
							
//							console.log(getAllShopCar[itemKey].itemGuige[i].iGAGAllGuige.split("、"))
//							getAllShopCar[itemKey].itemGuige[i].iGAGAllGuige.split("、").forEach((oneStyleItem,osIndex)=>{
//								console.log(DB.getItem("allGoodsAttrs").toJson())
//								DB.getItem("allGoodsAttrs").toJson().forEach((agaItem,agiIndex)=>{
//										if (oneStyleItem == agaItem.attrName) {
//											oneStyleItemArr.push(agaItem);
//										}									
//								})
//							})
							
							pushGoods.listTeaOrderDetailsAttr = oneStyleItemArr;
							listTeaOrderDetails.push(pushGoods)
						}
					}
				}
				console.log("listTeaOrderDetails")
//				console.log(listTeaOrderDetails)
				
				var toPushData = {
					userNo:telUserNo.userNo,
					telephone:telUserNo.telephone,
					promotionId:this.promotionId,			//活动参数
					remark:this.textAreaValue, //备注
					orderType:1,				//1堂吃2预约
					storeNo:DB.getItem("storeNo").toString(),
					listTeaOrderDetails:listTeaOrderDetails  //商品和其规格
				};
				
				console.log("toPushData")
//				console.log(toPushData)
//				console.log(toPushData.listTeaOrderDetailsAttr)
				
				this.$http.post("/userOrderInfo/userOrderOper",toPushData).then((res) => {
					console.log("/userOrderInfo/userOrderOper")
					if(res.status == 200) {
						if(DB.getItem("localLang").toString() == "en"){
							var ErrorMsg = res.data.usErrorMsg;
						}else{
							var ErrorMsg = res.data.cnErrorMsg;
						}						
						if(res.data.rspCode == "00000"){
							console.log(res.data.data)
							this.initOrderPage(res.data.data);
							this.orderNo = res.data.data.orderNo;
							this.goodsId = res.data.data.listTeaOrderDetails[0].goodsId;
							this.showInitData = true;
						}else{
     		               		this.$vux.toast.show({
									text: ErrorMsg,
									type: "text",
								})							
							this.$router.openPage("/classification");
						}
					}
				}).catch((err) => {
					console.log(err)
				})	    	
	    },
	    //初始化页面数据
	    initOrderPage(value){
				var getAllShopCar = this.shopCar.getAll();
				var allGoodsDataPage = {allGDPage:[],allGDLength:value.listTeaOrderDetails.length,allGDOrigPrice:value.origPrice,allGDDiscount:value.discount,allGDOrderPrice:value.orderPrice};
						    	
				for (var itemKey in getAllShopCar) {
						getAllShopCar[itemKey].itemGuige.forEach((item,i)=>{
						var allGDPageObj = {};
						allGDPageObj.goodsItem = {};
						allGDPageObj.goodsItem.goodsName = getAllShopCar[itemKey].goodsItem.goodsName;
						allGDPageObj.goodsItem.goodsPrice   = getAllShopCar[itemKey].goodsItem.goodsPrice;
						allGDPageObj.goodsItem.goodsGuigePrice  = getAllShopCar[itemKey].goodsItem.goodsGuigePrice;
						allGDPageObj.goodsItem.goodsPictureRound = getAllShopCar[itemKey].goodsItem.goodsPictureRound;
						allGDPageObj.goodsItem["goodsGuigePrice"] = item.guiGePrice;
						allGDPageObj.iGAGAllGuige = item.iGAGAllGuige;
						allGDPageObj.itemOneGuigeLen = item.itemOneGuigeLen;
						allGDPageObj.hasGuigePrice = item.hasGuigePrice;
						
						allGoodsDataPage.allGDPage.push(allGDPageObj)
				})
				}
				console.log("allGoodsDataPage")
				console.log(allGoodsDataPage)
				this.allGoods = allGoodsDataPage;
				this.showLoading = false;
	    		

	    		
	    },
	    //更改订单
	    finishPayModfiyOrder(){
		    	if (this.isShowOrderTime) {
		    		var theOrderTime = formatTime.ftNoHMS(new Date()) +" "+ this.formatDemoValue[0] + ":" + this.formatDemoValue[1] + ":00";
		    	} else{
					var theOrderTime = "";	    		
		    	}
		    	console.log("theOrderTime")
		    	console.log(theOrderTime)
		    	var data = {
		    		orderTime:theOrderTime,
					orderNo:this.orderNo,
					remark:this.textAreaValue,
				}
	    	
				this.$http.get("/userOrderInfo/finishPayModfiyOrder",{params:data}).then((res) => {
					console.log("/userOrderInfo/finishPayModfiyOrder")
					console.log(res.data)
					if(res.status == 200 && res.data.rspCode == "00000") {
						
					}
				}).catch((err) => {
					console.log(err)
				})		    	
	    	
	    	
	    },
	    //提交订单
	    updateAccount(){
	    	var telUserNo = DB.getItem("telUserNo").toJson();
			console.log("telUserNo")
			console.log(telUserNo)
	    	var _this = this;
			if(_this.allGoods.allGDOrderPrice){
				_this.finishPayModfiyOrder();
				//根据不同设备和选择的支付方式支付
				if (this.valueDefault[0] == "VISA/Master/Debit") {
	  				this.stripePay();
				} else if(this.isPcMoWx == "wx"){
					var openId = JSON.stringify({openId:telUserNo.weixinOpenid});
					_this.wPayData("WX_JSAPI",openId);
//					_this.wPayData("WX_JSAPI",`{"openId":"oO3y_1Ezzso4gsJOTK4UsX1zE5Uw"}`);
//					this.wxPay();
				} else if(this.isPcMoWx == "pc"){
					if(this.valueDefault[0] == "支付宝"){
						_this.pcAliWxPayData("ALIPAY_QR","{}");
					}else if(this.valueDefault[0] == "微信支付"){
						_this.pcAliWxPayData("WX_NATIVE",`{productId:${_this.goodsId}}`);
					}
				} else if(this.isPcMoWx == "mo"){
					var openId = JSON.stringify({openId:telUserNo.weixinOpenid});
					_this.wPayData("WX_JSAPI",openId);					
				}
			
			}
	    	
	    },
	    
	    //微信支付
	    wxPay(){
	    	var telUserNo = DB.getItem("telUserNo").toJson();
	    	var data = {
	    		orderNO:this.orderNo,
	    		userId:telUserNo.userNo
	    	}
			this.$http.post("/wxpay/pay",data).then((res) => {
				console.log("/wxpay/pay")
				console.log(res.data)
				if(res.status == 200 && res.data.rspCode == "00000" && res.data.data) {
					console.log(res.data.data)
					onBridgeData = res.data.data;
					if(onBridgeData.paySign){
//						onBridgeReady();	
						this.onBridgeReady();
					}else{
						this.$vux.toast.show({
							text: "微信支付调用失败！",
							type: "text",
						})				
					}					
				}
			}).catch((err) => {
				console.log(err)
			})	
	    },	    

	    //微信支付
	    onBridgeReady(){
	    	var configData = {};
	    	var strUrl = location.href.split('#')[0];
			console.log(strUrl)
			this.$http.get("/getconfig",{
				params:{
					pageUrl:strUrl
				}
			}).then((res) => {
				console.log("/getconfig")
				console.log(res.data.data)
				if(res.status == 200 && res.data.rspCode == "00000" && res.data.data) {
					console.log(res)
					configData = res.data.data;
					this.configData(configData);
				}
			}).catch((err) => {
				console.log(err)
			})		    	
	    },
	    
 		//config 
	    configData(configData){
	    			var _this = this;
					console.log("configData")
	       			console.log(configData)	    	
					wx.config({
					    debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
					    appId: configData.appId, // 必填，企业号的唯一标识，此处填写企业号corpid
					    timestamp:configData.timestamp , // 必填，生成签名的时间戳
					    nonceStr: configData.nonceStr, // 必填，生成签名的随机串
					    signature: configData.signature,// 必填，签名，见附录1
					    jsApiList: ["openAddress","getLocation","chooseWXPay"], // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
					});

					wx.ready(function(res){							
						console.log("wxConfig-success")
						console.log(res)
							//支付
							console.log("onBridgeData")
							console.log(onBridgeData)										
							wx.chooseWXPay({  
			                  timestamp: onBridgeData.timestamp, // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符  
			                  nonceStr: onBridgeData.nonceStr, // 支付签名随机串，不长于 32 位  
			                  package: onBridgeData.package, // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）  
			                  signType: onBridgeData.signType, // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'  
			                  paySign: onBridgeData.paySign, // 支付签名  
			                  success: function(res) {  
									console.log("chooseWXPay-success")
									console.log(res)
			                      // 支付成功后的回调函数  
			                      if (res.errMsg == "chooseWXPay:ok") {  
			                          //支付成功  
				   					_this.$vux.toast.show({
					    			 type:'text',
									 text:"微信付款成功！"
									})
				   					_this.$router.openPage("/mineOrder");
				   					
			                      } else {  
				                      _this.$vux.toast.show({
						    			 type:'text',
										 text:res.errMsg
										})
			                      }  
			                  },  
			                  cancel: function(res) {  
			                      //支付取消  
			                      console.log('支付取消')
			                  }  
			              });
			              
					})
		
			wx.error(function(res){
			    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
				console.log("wxConfig-error")
				console.log(res)
			
			});		    	
	    },	    
	    
	    //通用
	    //敬请期待
	    nextWait(){
	      this.showPositionValue = true;
	    },
	    changeChecker(value){
	    	console.log(value.key)
			if (value.key == 0) {
				this.isShowOrderTime = true;
				this.checkerDemo = this.demo2;
			}else{
				this.isShowOrderTime = false;
				this.checkerDemo = this.demo1;
			}	    	
	    }
	}
	
	
}
	
	
</script>

<style lang="scss" type="text/scss" scoped="scoped">
@import "../../../assets/scss/util";
.ca-cont{
  background: linear-gradient(#FDA544 0%,#FFF 60%); /* 标准的语法 */
	
.classification-footer {
  left: 0;
  right: 0px;
  bottom: 0px;
  /*top: calc(100vh - 1.4rem);*/
  height: $footerHeight;
  display: flex;
  z-index: 333;
  .cf-left {
    flex: 1;
    background-color: rgb(80, 80, 83);
    display: flex;
    justify-content: center;
    align-items: center;
    .cfl-cont {
      width: 1.2rem;
      height: 1.2rem;
      text-align: center;
      position: relative;
	  background-color:  rgb(80, 80, 83);
      top: 0.1rem;
      .ftl-gwc {
      	border-radius: 50%;
        height: 1rem;
        color: #fff;
      }
      .ftl-redPoint {
        width: 0.4rem;
        height: 0.4rem;
        border-radius: 0.4rem;
        font-size: 0.3rem;
        color: #fff;
        background-color: #ff0000;
        position: absolute;
        top: 0;
        right: 0;
      }
    }
  }
  .cf-center {
    flex: 3;
    background-color: rgb(80, 80, 83);
    color: #fff;
    line-height: $footerHeight;
  }
  .cf-right {
    flex: 1.5;
    background-color: #FDA544;
    color: #fff;
    text-align: center;
    line-height: $footerHeight;
  }
}

/*中间*/
.closeAccountCont{
	padding: 0.4rem ;
	width: 9.2rem;
	top: 1.2rem;
	bottom:  $footerHeight;
	overflow-y: auto;
		.groupCont{
			.tabItem{
				color: #FDA544;
			}
	}
	.goodsDetail{
		margin-top: 0.4rem;
		background-color: #FFF;
		.gd-title{
			font-size: 0.44rem;
			border-bottom: 1px solid #DDD;
			height: 1.2rem;
			line-height: 1.2rem;
			padding-left: 0.4rem;

		}
		.gd-cont{
			.gdc-detail{
					height: 2rem;
					display: flex;
					font-size: 0.32rem;
				.gdcd-img{
					width: 1.6rem;
					height: 1.6rem;
					padding: 0.2rem;
					/*border-radius: 50%;*/
				}	
				.gdcd-price{
					width: calc(10rem - 2.4rem);
					height: 1rem;
					line-height: 1rem;
					display: flex;
					justify-content: space-between;		
					color: #666;
					.gdcdp-right{
						display: flex;
						flex-direction: column;
						div{
							height: 0.5rem;
						}
						.gdcdpr-title{
							height: 0.6rem;
							font-size: 0.42rem;
							color: #333;
						}

					}
					.gdcdp-plus{
						display: flex;
						flex-direction: column;
						justify-content: space-between;		
						padding-right: 0.6rem;		
						font-size: 0.38rem;		
						span{
							display: inline-block;
							height: 0.6rem;
						}
					}			
				}			
			}
		}
		.gdc-footer{
				color: #333;
				padding:0  0.2rem 0.2rem 0.2rem ;
			.gdcf-allPrice{
				padding-right: 0.6rem;				
				padding-bottom: 0.2rem;				
				text-align: right;
				.fl{
					float: left;
					padding-left: 0.1rem;	
				}
			}
			.gdc-textarea{
				border: 1px solid #D0D6D6;
			}
		}
	}
	}
	}

.demo1-item {
  border: 1px solid #ececec;
  margin-right: 0.2rem;
  padding: 0.06rem 0.08rem;
  font-size: 0.38rem;
}
.demo1-item-selected {
  /*border: 1px solid #FDA544;
  background-color: #FDA544;
  color: #FFF;*/
   border: 1px solid #000;
  background-color: #000;
  color: #FFF; 
  
}
.demo1-item-disabled,.disabled{
  border: 1px solid #999;
  background-color: #FFF;
  color: #999;
  opacity: 0.2;
}


.dialogGoodsDet {
  .weui-dialog{
    border-radius: 8px;
    padding-bottom: 8px;
  }
  .img-box {
  	max-height: 9rem;
    overflow-y:auto;   	
  	img{
  		height: 6rem;
  	}
  	.dgdCont{
  		font-size: 0.3rem;
  		text-align: center;
  		padding-top: 0.2rem;
  		color: rgba(255,0,0,0.9);
  	}
  }
}

</style>