
玩家支付流程
------------
托管模式下，玩家点击购买按钮，触发支付行为，SDK会根据玩家支付的币种动态选择出相应的支付渠道列表和从服务器端设置好的物品包价格，玩家可以选择自己需要的物品包并通过合适的渠道进行支付，玩家付款之后，支付平台会向应用提供的回调地址发送通知，同时前端获取支付结果。

代理模式下，由应用构造出订单传递给SDK，SDK为玩家弹出付款页面，玩家付款之后，支付平台会向应用提供的回调地址发送通知，同时前端获取支付结果。

准备工作
--------
在准备计入之前首先需要从运营方获获取appid,appid作为接入系统的标识，是一个单独的统计单元，且都有各自的配置。一个项目下允许有多个appid。

应用需要提供一个回调地址接收支付结果的通知，对于单机版无服务端的应用可以忽略这项配置。但对于需要联网的应用建议都采用服务端对服务端的通知模式，以确保交易安全。具体的回调地址规则请参见服务端回调验证文档。

当您获取了appid，并设置了回调地址之后就可以开始集成支付组件了。

在应用中引入lib工程
-------------------
请参照快速集成中（引入337lib工程）

在Manifest文件添加必要的声明
----------------------------
	
需要添加内容入下: ::

	<uses-permission android:name="android.permission.INTERNET"/>
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
	<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
	<uses-permission android:name="com.android.vending.BILLING" />
	<uses-permission android:name="android.permission.RECEIVE_SMS" />
	<uses-permission android:name="android.permission.SEND_SMS" />
	<uses-feature android:name="android.hardware.telephony" android:required="false"></uses-feature>

	<activity 
	android:name="com.web337.android.pay.PayCoreMobileActivity" 
	android:configChanges="orientation|keyboardHidden|screenSize"/>

	<activity 
	android:name="com.web337.android.pay.PayShowPackagesActivity" 
	android:configChanges="orientation|keyboardHidden|screenSize" />

	<activity 
	android:name="com.fortumo.android.FortumoActivity" 
	android:theme="@android:style/Theme.Translucent.NoTitleBar" />

	<activity 
	android:name="com.web337.android.pay.fortumo.FortumoActivity" 
	android:theme="@android:style/Theme.Translucent.NoTitleBar" 
	android:configChanges="orientation|keyboardHidden|screenSize" />
	
	<receiver android:name="com.fortumo.android.BillingSMSReceiver" >
		<intent-filter>
			<action android:name="android.provider.Telephony.SMS_RECEIVED" >
			</action>
		</intent-filter>
	</receiver>

	<service android:name="com.fortumo.android.FortumoService" />
	<service android:name="com.fortumo.android.StatusUpdateService" />
	
初始化PayCore
-------------

通过调用 ``PayCore.init(Context context,String appid,PayCallback back)`` 方法来进行初始化。
    
第一个参数Context,传递当前的activity即可。
    
第二个参数appid，要传应用获得的appid。
    
第三个参数PayCallback，这是一个前端回调类，因为SDK初始化和支付是异步进行的，所以并不能立即获取相应结果。应用需要监听这些回调方法，并作出相应的处理。该接口共有四个方法，分别如下：
    
      ``onInitFinish(Msg msg)``          初始化完成
		 
      ``onComplete(Order o)``          完成订单
		 
      ``onCancel()``               		订单被取消
		 
      ``onFailed(Msg msg)``				支付失败	

支付准备
--------

必须确保当前已经为支付模块设置过uid才能发起支付。

设置uid有以下几种方法：

#. 使用Settings.setCommonUid,该方法需要在支付初始化之前设置，设置之后，支付模块在初始化时会使用该方法设置的uid。（在引入IdZone之后这个方法已经过时，只用于兼容旧版代码和单独接入支付时使用）
 
#. 使用337用户id，若未使用Commonuid，若在支付初始化之前已经登录了337uid，则支付模块在初始化时会使用用户的337uid。
 
#. 发起支付前设置，使用PayCore.setUid(id)方法，可以在发起支付前设置uid。
 
设置uid之后，还可以设置角色id，PayCore.setRoleId(roleid)方法可以单独设置角色id，若未设置，则角色id将和uid相同。

IdZone设置： 2.0.4版本后增加了IdZone的概念，使用IdZone的前提是使用337登录模块。在登录完成之后，手动调用IdZone的方法设置角色名、角色ID、服名称和服ID，设置完成后，支付系统会默认使用用户337uid和角色id，不再支持自定义uid。客服系统也将使用IdZone中的角色名和服名称

设置方法如下: ::

	com.web337.android.id.Zone.getInstance().setRole_id("12345");
	com.web337.android.id.Zone.getInstance().setRole_name("王小明");
	com.web337.android.id.Zone.getInstance().setServer_id("1");
	com.web337.android.id.Zone.getInstance().setServer_name("琉璃仙境");	  
	  
发起支付
--------

* 代理模式:

 在代理模式下，SDK只负责按照应用内预定的方式引导玩家到支付渠道进行付款，并及时反馈给客户端支付结果。

 代理模式下，部分第三方的支付方式需要手动添加，比如台湾大哥大和GooglePlay应用内购

 添加台湾大哥大

代码如下: ::

	if (PayCore.add(PayCore.SDK_TWM)) {
		PayCore.twm.bind("应用在大哥大处申请到的支付代码", "当前购买的商品在应用内部的id");
		PayCore.twm.init(Context context);
	}
	
添加GooglePlay内购

代码如下： ::

	if (PayCore.add(PayCore.SDK_GOOGLEPLAY)) {
		PayCore.googleplay.bindSKU("应用在google申请的内购代码", "当前购买的商品在应用内部的id");
		PayCore.googleplay.init(new initGooglePlayListener() {
			@Override
			public void initSuccess() {
			}

			@Override
			public void initFailed(String msg) {
			}
		});
	}
	
发起支付: ::

	beginPay(Context c,Order o)
	
第一个参数传递当前的activity即可。

第二个参数order需要是com.web337.android.model.Order的实例，发起支付时必需的属性如下：

amount，传递商品数量。

description，传递商品描述，比如10个元宝、100枚金币等，会显示在第三方渠道的支付页面上。

gross，要支付的金额。对于Google Play内购支付来说，玩家的真实花费和该值无关系，支付平台会回调的金额是所传的金额，而对于第三方支付比如paypal，真实花费就是所传的金额。举例说明，一件商品在Google Play上的内购价格为0.99美元，发起支付时gross设置为0.99，当香港玩家使用Google Play内购时，所花费的是0.99美元换算成港币的金额，而使用paypal支付时，必须要去支付0.99美元。之后支付平台会回调的金额还是0.99美元。

currency，支付的货币类型，该值和gross共同起作用，使用ISO-4217标准货币代码，如USD(美元)、TWD(新台币)等。

productId，应用自定义的商品代码，通常应用对于特定的商品都会有特定的代码，比如一个关卡、一组金币、一个新功能等，这个值是为了方便游戏识别用户所购商品，在使用Google Play内购支付时，需要将商品代码和应用在Google Play内购代码进行绑定，SDK会根据所传的productId来获取真正的内购代码，这样应用在发起支付时，就无需区分是用Google Play内购还是第三方渠道进行支付了，同时应用的服务端接受回调时，也无需区分，只需要识别productId即可。

customData，自定义参数，应用可以随意传递任何数据，长度为200。支付平台会将该值原样回调。应用可以自行决定如何使用该值。该值不能为空字符串。

* 托管模式

在托管模式中不在需要自己手动添加和绑定台湾大哥大和Google Play两个支付渠道,SDK会根据后台提供的配置自己进行初始化和绑定工作。

应用直接调用以下方法即可发起支付 ::

	PayCore.show();
	
SDK会直接展示在后台预设好的物品包金额，从而方便用户进行快速支付。
		
*两种支付模式是并列的，不能同时使用。代理模式一般需要将需要购买的物品金额等信息配置在应用内部，然后作为参数进行支付，而托管模式全部参数都在服务器端配置，可以灵活的调整物品包的金额种类等*

其他说明
--------

单机版无服务端的应用可以通过PayCallback来获取支付结果，这部分的回调可能会有一些延迟。

