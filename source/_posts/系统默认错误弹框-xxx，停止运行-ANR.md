layout: post
title: 屏蔽系统默认错误弹框--xxx，停止运行/ANR
comment: true
tags: [Android,Framework, 技术]
date: 2016-08-27 19:21:28
updated: 2016-08-27 19:21:28
---

------

因为运营商某些apk的兼容性存在一定程度问题，但apk因为特殊体制的问题，不会对其apk进行更改；这时就需要底层协助厂商处理一些兼容性的apk，比如某些时候apk退出会弹出一个‘xxx，已停止运行’，但实际上apk所有功能均正常，不影响用户的实际体验，于是乎就去想办法去掉这个ErrorDialog；
依旧根据常规分析思路，现在找到其对应的source code：
  <!--more-->
#### 1.根据“已停止运行”找到关键字;
`frameworks/base/core/res/res/values-zh-rCN/strings.xml`
![error1](http://oa1wnpe3m.bkt.clouddn.com/Selection_001.png)
```xml
 <string name="aerr_application" msgid="932628488013092776">"很抱歉，“<xliff:g id="APPLICATION">%1$s</xliff:g>”已停止运行。"</string>
    <string name="aerr_process" msgid="4507058997035697579">"抱歉，进程“<xliff:g id="PROCESS">%1$s</xliff:g>”已停止运行。"</string>
    <string name="anr_title" msgid="4351948481459135709"></string>
    <string name="anr_activity_application" msgid="1904477189057199066">"<xliff:g id="APPLICATION">%2$s</xliff:g> 无响应。\n\n要将其关闭吗？"</string>
    <string name="anr_activity_process" msgid="5776209883299089767">"<xliff:g id="ACTIVITY">%1$s</xliff:g> 活动无响应。\n\n要将其关闭吗？"</string>
    <string name="anr_application_process" msgid="8941757607340481057">"<xliff:g id="APPLICATION">%1$s</xliff:g> 无响应。要将其关闭吗？"</string>
    <string name="anr_process" msgid="6513209874880517125">"<xliff:g id="PROCESS">%1$s</xliff:g> 进程无响应。\n\n要将其关闭吗？"</string>
    <string name="force_close" msgid="8346072094521265605">"确定"</string>
    <string name="report" msgid="4060218260984795706">"报告"</string>
    <string name="wait" msgid="7147118217226317732">"等待"</string>

```
#### 2. 根据aerr_application 查找对应的dialog
 `/frameworks/base/services/java/com/android/server/am/AppErrorDialog.java`
![e2](http://oa1wnpe3m.bkt.clouddn.com/Selection_002.png)
![e3](http://oa1wnpe3m.bkt.clouddn.com/Selection_003.png)
```java
 public AppErrorDialog(Context context, ActivityManagerService service,
            AppErrorResult result, ProcessRecord app) {
        super(context);
        
        Resources res = context.getResources();
        
        mService = service;
        mProc = app;
        mResult = result;
        CharSequence name;
        if ((app.pkgList.size() == 1) &&
                (name=context.getPackageManager().getApplicationLabel(app.info)) != null) {
            setMessage(res.getString(
                    com.android.internal.R.string.aerr_application,
                    name.toString(), app.info.processName));
        } else {
            name = app.processName;
            setMessage(res.getString(
                    com.android.internal.R.string.aerr_process,
                    name.toString()));
        }

        setCancelable(false);
}
```
#### 3. 根据AppErrorDialog找到调用dialog的地方AMS，修改AMS的控制源头即可控制是否弹出对话框
![e4](http://oa1wnpe3m.bkt.clouddn.com/Selection_004.png)
  `/frameworks/base/services/java/com/android/server/am/ActivityManagerService.java`
 
 ```java
 
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
            case SHOW_ERROR_MSG: {
                HashMap<String, Object> data = (HashMap<String, Object>) msg.obj;
                boolean showBackground = Settings.Secure.getInt(mContext.getContentResolver(),
                        Settings.Secure.ANR_SHOW_BACKGROUND, 0) != 0;
                synchronized (ActivityManagerService.this) {
                    ProcessRecord proc = (ProcessRecord)data.get("app");
                    AppErrorResult res = (AppErrorResult) data.get("result");
                    if (proc != null && proc.crashDialog != null) {
                        Slog.e(TAG, "App already has crash dialog: " + proc);
                        if (res != null) {
                            res.set(0);
                        }
                        return;
                    }
                    if (!showBackground && UserHandle.getAppId(proc.uid)
                            >= Process.FIRST_APPLICATION_UID && proc.userId != mCurrentUserId
                            && proc.pid != MY_PID) {
                        Slog.w(TAG, "Skipping crash dialog of " + proc + ": background");
                        if (res != null) {
                            res.set(0);
                        }
                        return;
                    }
                    if (mShowDialogs && !mSleeping && !mShuttingDown&&SystemProperties.getBoolean("mstar.app.compatibility.enable", false)) {
                        Dialog d = new AppErrorDialog(mContext,
                                ActivityManagerService.this, res, proc);
                        d.show();
                        proc.crashDialog = d;
                    } else {
                        // The device is asleep, so just pretend that the user
                        // saw a crash dialog and hit "force quit".
                        if (res != null) {
                            res.set(0);
                        }
                    }
                }

                ensureBootCompleted();
            } break;
            }
 ```
 
   在上述代码中加入了兼容性开关的一个控制`SystemProperties.getBoolean("mstar.app.compatibility.enable", false)`，这样就不必再担心某些厂商的apk兼容性测试做的不好，从而导致一些用户体验的问题。
   
