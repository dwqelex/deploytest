

概述
----

悬浮窗功能旨在不破坏原有游戏界面的情况下，为用户提供进行常用功能快捷操作的部件。同时悬浮窗功能只在游戏内使用，不需要额外声明。

在应用中引入lib工程
-------------------
请参照快速集成中（引入337lib工程）

使用悬浮窗
----------

* 显示悬浮窗

 调用 ``FuncCore.showFloatWindow(Activity activity)`` 即可显示默认的悬浮窗。目前默认的悬浮窗有四个功能分别是帮助(用户反馈)，充值，粉丝页（如facebook等），论坛。
	
 调用 ``FuncCore.showFloatWindow(Activity activity,View... ibs)`` 自定义悬浮窗的内容
	
* 隐藏悬浮窗

 调用 ``FuncCore.destoryFloatWindow(Activity activity)`` 隐藏悬浮窗