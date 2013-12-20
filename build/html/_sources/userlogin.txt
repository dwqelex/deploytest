

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
	<uses-permission android:name="android.permission.GET_ACCOUNTS"/>

	<activity 
	android:name="com.web337.android.user.UserPage" 
	android:theme="@style/mobile337user"/>

	<activity 
	android:name="com.facebook.LoginActivity" 
	android:theme="@android:style/Theme.Translucent"/>
	
	<activity
	android:name="com.web337.android.user.GoogleAcountLogin"
	android:configChanges="orientation|keyboardHidden"/>

	<meta-data 
	android:name="com.facebook.sdk.ApplicationId" 
	android:value="\ 156198457906087"/>

	<activity 
	android:name="com.web337.android.widget.Web" 
	android:configChanges="orientation|keyboardHidden|screenSize" 
	android:launchMode="singleTask"/>

获取用户信息
------------

应用可以通过直接调用 ``UserCore.checkLogin(Context c,UserLoginCallback callback)`` 然后在 ``UserLoginCallback`` 中获取用户信息。而不需要关心是否注册等细节。在 ``UserLoginCallback`` 接口中有两个方法
	 
* ``onLoginSuccess(User u,boolean isRegist)`` 
通过此回调回去用户信息，如果isRegist为true则此用户为注册用户，如果为false则此用户为登录用户
	
* ``public void onCancel()``
如果用户在弹出的登录-注册界面没有登录或者注册，则回调此方法
	
自定义用户登录界面
------------------

如果您希望使用个性化的登录界面，你有以下两种选择：
		
* 直接更改SDK提供的登录界面布局文件 ``/res/layout/mobilev2_337_user_login.xml`` ，需要注意的是您可以任意更改此布局文件的样式，但是请务必保证每个组件的功能和ID不要变更。
	
* 完全使用自己的布局文件。然后调用SDK中的接口来使用相应功能，可以调用的方法请参照下一节

其他方法
--------
#. 获取当前登录用户：

   ``UserCore.getLoginUser();``
   
#. 退出登录：

   ``UserCore.logout();``

#. 获取用户登录状态：

   ``UserCore.isLogin();``

#. 获取最后一次登录的用户名：

   ``UserCore.lastLoginUsername();``

#. 找回用户密码：

   ``UserCore.getPassword(Context c);``

#. 使用用户名和密码登陆：

   使用前需要先调用setContext(Context c)方法

   ``login(final String username,final String password,final UserSelfLoginCallback callback);``

#. 注册用户：
	
   使用前需要先调用setContext(Context c)方法
	
   ``register(final String username,final String password,final String email,final UserSelfRegisterCallback callback);``

#. 检查用户是否合法：

   ``check(final User u,final UserSelfCheckCallback callback);``

#. 个人信息页：


 调用方法打开页面：

 ``UserCore.showUserInfo(Activity);``
	
 当前无337登录用户时，方法返回false，当前有337登录用户时，方法返回true，并打开用户个人信息页。玩家可以在此页面进行更改个人信息、更改密码、切换账户等操作。

 如果玩家在个人信息页切换了账户，当玩家关闭个人信息页时，SDK会将新的用户回调给开发者。回调方法的设置如下： ::

	UserCore.setOnChangeUserListener(new com.web337.android.user.UserCore.OnChangeUserListener(){
	 @Override
	 public void onChange(User u) {
		if(u != null){
			alert("更改用户："+u.getUsername());
		}else{
			alert("退出登录");
		}
	}});

