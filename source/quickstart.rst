===============				
快速集成
===============
That has a paragraph about a main subject and is set when the '='
is at least the same length of the title itself.

引入337lib工程
--------------

	.. image:: _static/includelib.jpg


添加权限  
-------- 
	
需要声明的权限如下: ::
	
	<uses-permission android:name="android.permission.INTERNET"/>
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
	<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
	<uses-permission android:name="com.android.vending.BILLING" />
	<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
	<uses-permission android:name="android.permission.RECEIVE_SMS" />
	<uses-permission android:name="android.permission.SEND_SMS" />
 
添加Activity、MetaData和其他内容 
--------------------------------
需要添加的内容如下: ::

		<activity 
			android:name="com.web337.android.user.UserPage" android:theme="@style/mobile337user"/>
		<activity 
			android:name="com.web337.android.pay.PayCoreMobileActivity" 
			android:configChanges="orientation|keyboardHidden|screenSize"/>
		<activity 
			android:name="com.web337.android.pay.PayFromPCActivity" 
			android:configChanges="orientation|keyboardHidden|screenSize"/>
		<activity 
			android:name="com.web337.android.pay.PayShowPackagesActivity" 
			android:configChanges="orientation|keyboardHidden|screenSize" />
		<activity 
			android:name="com.facebook.LoginActivity" android:theme="@android:style/Theme.Translucent"/>
		<meta-data android:name="com.facebook.sdk.ApplicationId" android:value="\ 220782057940018"/>
		
		<activity 
			android:name="com.fortumo.android.FortumoActivity" 
			android:theme="@android:style/Theme.Translucent.NoTitleBar">
		</activity>
		<activity 
			android:name="com.web337.android.pay.fortumo.FortumoActivity" 
			android:theme="@android:style/Theme.Translucent.NoTitleBar" 
			android:configChanges="orientation|keyboardHidden|screenSize">
		</activity>
		<activity 
			android:name="com.web337.android.widget.Web" 
			android:configChanges="orientation|keyboardHidden|screenSize" 
			android:launchMode="singleTask">
		</activity>
		<activity 
			android:name="com.web337.android.ticket.TicketCoreActivity" 
			android:configChanges="orientation|keyboardHidden|screenSize" >
		</activity>
		<receiver android:name="com.fortumo.android.BillingSMSReceiver">
		  <intent-filter>
		    <action android:name="android.provider.Telephony.SMS_RECEIVED"></action>
		  </intent-filter>
		</receiver>
		
		<uses-feature android:name="android.hardware.telephony" android:required="false"></uses-feature>
		
*screenSize添加时如果出现错误，请更改targetSdkVersion为13以上即可*	

添加广告SDK配置
---------------

详见广告SDK接入文档

初始化所有第三方SDK
-------------------
	 
直接调用一下方法:  ::
	 
		com.web337.android.sdks.SdkCore.initAll(this);
		
接入用户系统
------------
*  实例化一个回调对象： ::
	
		final UserLoginCallback callback = new UserLoginCallback(){
			@Override
			public void onCancel() {
				alert("取消登录");
			}

			@Override
			public void onLoginSuccess(User user, boolean isregister) {
				/*
				登录成功后设置角色id、角色名称、服id、服名称
				*/
				com.web337.android.id.Zone.getInstance().setRole_id("roleid00001");
				com.web337.android.id.Zone.getInstance().setRole_name("wangxiaoming");
				com.web337.android.id.Zone.getInstance().setServer_id("1");
				com.web337.android.id.Zone.getInstance().setServer_name("ServerName");
				//成功后可以显示一个欢迎提示，玩家可以在此切换账号
				UserCore.showWelcome(YourActivity.this);//【可选功能】
				if(isregister){
					alert("注册成功"+user.getUid());
				}else{
					alert("登录成功"+user.getUid());
				}
			}
		};
		
*  调用checkLogin方法： ::

		com.web337.android.user.UserCore.checkLogin(Context c, UserLoginCallback callback)

*可以在进入游戏主页面后直接调用，Context传递当前的activity即可，UserLoginCallback传递上一步创建的callback对象*

**调用该方法后只需要关心callback中的两个回调方法即可，若当前无登录用户，则会弹出登录或注册页面，用户登录或注册完成后，会回调。若已经有登录的用户，则直接回调**

发送行云统计
------------

用户登录完成后，设置完角色信息即可调用: ::

	com.web337.android.sdks.XA.send(Context c);
	
Context传递当前Activity即可
			
支付初始化
----------

示例代码如下: ::
	
		String appid = "xxx@android_tw_1";
		PayCore.init(this, appid, new PayCallback(){
			@Override
			public void onCancel() {
				alert("取消支付");
			}

			@Override
			public void onComplete(Order o) {
				/*
				 * 支付完成表明玩家付款成功，但并不表明支付中心回调游戏服务端是成功的
				 */
				alert("支付完成。"+o.getTransid());
			}

			@Override
			public void onFailed(Msg msg) {
				alert("支付失败。"+msg.getMsg());
			}

			@Override
			public void onInitFinish(Msg msg) {
				if(msg.isSuccess()){
					alert("初始化完成。");
				}else{
					alert("初始化出错。"+msg.getMsg());
				}
		}});
	
**初始化涉及网络操作，不建议将此步骤放在用户点击商城的时候**

展示套餐
--------

需要如下三个步骤: ::

		//若未设置过commonUid,则需要设置uid才能去支付
		//PayCore.setUid("elex337_12345");
				
		//设置角色id，回调时会回调此id，若未设置，则角色id为uid
		PayCore.setRoleId("abcde12345");
				
		//展示套餐，此处套餐均在支付平台后台配置
		PayCore.show();

***至此337 SDK接入完成，更多的使用方法可以参考各个模块的文档***


 