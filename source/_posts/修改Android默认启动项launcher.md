layout: post
title: 修改Android默认启动项launcher
comment: true
tags: [技术, Framework, Android]
date: 2016-08-27 19:19:02
updated: 2016-08-27 19:19:02
---
#### 问题背景：
------
因为目前很多IPTV的厂商+广电的要求，不允许用户自己替换自己的launcher，为了保证利益，强行推广自己的launcher，对播控平台的掌控，于是就必须要求芯片原厂提供turnkey方案时符合其要求。下面用Android 5.1.1来举例说明，理论上AN的版本对此模块改动差异不太多。

#### Google官方Android原生Launcher设置
Launcher是Android系统的桌面、是android系统的主要组件。android系统允许存在多个Launcher并设置默认主界面。

应用程序作为Home(主界面)需在Activity的intent-filter节点中添加如下内容
```xml
<category android:name="android.intent.category.HOME" />
<category android:name="android.intent.category.DEFAULT" />
```
当系统中存在多个Home app且没有设置默认，用户点击Home键会弹出如下图所示的界面（图一）： 
![launcher](http://oa1wnpe3m.bkt.clouddn.com/1.png)
用户可以选择“只有一次”或者“总是”来启动选择的APP
<!--more-->
一般情况下android中只会存在一个Home APP，系统启动后会直接启动此APP为默认，不需要用户选择。但是当系统中存在多个Home APP时，系统第一次启动就会弹出上图所示界面，让用户选择其中一个APP作为主屏幕应用，如果用户通过“始终”确认会设置选择的APP为默认的Home，用户通过“仅此一次”则此次以选择的APP为Home，再次按home键还是会弹出选择窗口.

#### 针对广电总局播控客制化方案
针对上图，思考一下，可以采用两种方案：
 1.  <font color=red>让google原生态中的选框自动选中并且自动进入，这样就符合我们的需求了</font>
 2. <font color=red>直接清除launcher选择的堆栈，让home的堆栈里面永远只保留我们需要的那个,这个是受Android原生code，AMS启动home activity启发</font>
 
#### 方案一
	根据操作流程分析code flow，两者互相糅合，理清问题的关键。
从上面弹出的选择默认主界面的界面入手，通过追踪发现上述界面是一个Activity，Activity代码路径frameworks/base/core/java/com/android/internal/app/ResolverActivity.java

>ResolverActivity分析
此Activity会获取系统中所有的Home app，并根据系统的设置情况显示如上界面。此类中有一个内部类ResolveListAdapter该类继承自BaseAdapter，该类是Home app选择界面的数据适配器。

ResolveListAdapter会在ResolverActivity的onCreate方法中被初始化并会传入一个ResolveInfo类型的List，ResolveListAdapter根据会传入的List<ResolveInfo>初始化一个List<DisplayResolveInfo> mList ，用户的点击事件都会在ResolveListAdapter获取数据。 
用户点击”ALWAYS”的事件发生在ResolverActivity的onButtonClick 方法中，此方法会获取选中的Item的position、或者获取用户上一次启动的Home app的，mAlwaysUseOption代表用户选中的是否为历史选择（如2图中的Launcher3），并调用startSelected。
```java
    public void onButtonClick(View v) {
        final int id = v.getId();
        startSelected(mAlwaysUseOption ?
                mListView.getCheckedItemPosition() : mAdapter.getFilteredPosition(),
  
                id == R.id.button_always,
                mAlwaysUseOption);
        dismiss();
    }
```
StartSeletced中通过ResolveListAdapter获取选择的item代表的Home app。并且finish此activity 
onIntentSelected会根据传入的ResolveInfo设置默认的Home，并根据Intent跳转到相应界面，onIntentSelected的代码在这里就不列出。
```java
    /**
     * 设置默认Home app并跳转，结束此Activity
     * @param which 用户选择的Item的position
     * @param always 是否设置为总是
     * @param filtered 是否非历史选择
     */

  void startSelected(int which, boolean always, boolean filtered) {
        if (isFinishing()) {
            return;
        }
         ResolveInfo ri = mAdapter.resolveInfoForPosition(which, filtered);
        Intent intent = mAdapter.intentForPosition(which, filtered);

        //设置默认Home，并启动
        onIntentSelected(ri, intent, always);
        finish();
    }
```
ResolveListAdapter的相关代码

```java
       /**
        *
        * @param position
        * @param filtered
        * @return
        */
        public ResolveInfo resolveInfoForPosition(int position, boolean filtered) {
            //mList为List<DisplayResolveInfo>
            return (filtered ? getItem(position) : mList.get(position)).ri;
        }
        /**
        *
        * @param position
        * @param filtered
        * @return
        */
        public Intent intentForPosition(int position, boolean filtered) {
            //mList为List<DisplayResolveInfo>
            DisplayResolveInfo dri = filtered ? getItem(position) : mList.get(position);
            return intentForDisplayResolveInfo(dri);
        }
```
**解决措施**
扯了半天了，分析完谷歌的原生流程，现在终于开始进行客制化的修改了。进入正题：
此Activity的onCreate方法中判断是否为第一次启动，如果是则调用startSelected方法设置默认Home app。

默认Home app的从ResolveListAdapter中获取，所以在ResolveListAdapter中添加`getDefaultHomePosition(String packageName)`方法，用于获取默认home app在List<DisplayResolveInfo> 中的位置，代码如下：
```java
public int getDefaultHomePosition(String packageName){
            for (int i = 0; i < mList.size(); i++) {
                ResolveInfo info = mList.get(i).ri;
                if (DEBUG)
                Log.w(TAG,"getDefaultHomePosition " + info.activityInfo.packageName);
                if (info.activityInfo.packageName.equals(packageName)) {
                   return i;
                }
            }
            return -1;
        }

```
在ResolverActivity中添加设置默认app的方法setupDefaultLauncher()，代码如下：

```java

 //用于记录默认home app是否设置过
    private static final String DEFAULT_HOME = "persist.sys.default.home";
    private void setupDefaultLauncher() {
        String first = "";
        try{
            first =  SystemProperties.get(DEFAULT_HOME);
        }catch(Exception e){
            Log.w(TAG,"exception error get DEFAULT_HOME");
        }
        //判断默认home 是否设置过，如果获取的字符串为空代表，未设置，否则return不在进行设置
        if (!TextUtils.isEmpty(first)) {
            return;
        }
        //使用包名获取所需设置的默认home app在ResolveListAdapter中的位置
        int position = mAdapter.getDefaultHomePosition("home app包名");
        //如果不存在则return
        if (position == -1) {
            if (DEBUG)
            Log.w(TAG,"not find default Home");
            return;
        }
        //设置默认home app后，则添加记录
        try{
            SystemProperties.set(DEFAULT_HOME,DEFAULT_HOME);
        }catch(Exception e){
            Log.w(TAG,"exception error set DEFAULT_HOME");
        }
        //设置默认home app，并跳转
        startSelected(position, true, true);
        //结束此activity
        dismiss();
    }
```
为了保证mAdapter被初始化 setupDefaultLauncher()的调用添加到ResolverActivity的onCreate函数中，代码如下：
```java
protected void onCreate(Bundle savedInstanceState, Intent intent,
            CharSequence title, int defaultTitleRes, Intent[] initialIntents,
            List<ResolveInfo> rList, boolean alwaysUseOption) {
        //其他初始化代码
        ............
        mIntent = new Intent(intent);
        mAdapter = new ResolveListAdapter(this, initialIntents, rList,
                mLaunchedFromUid, alwaysUseOption);
        //其它初始化代码
        ............

        if (mLaunchedFromUid < 0 || UserHandle.isIsolated(mLaunchedFromUid)) {
            // Gulp!
            finish();
            return;
        }

        int count = mAdapter.mList.size();
        //添加的代码
        setupDefaultLauncher();
        //原有逻辑
        //如果系统中home app大于1
        if (count > 1 || (count == 1 && mAdapter.getOtherProfile() != null)) {
            //初始化代码
            .........
        //如果home app等于1则设置唯一的home app为Home
        } else if (count == 1) {
            safelyStartActivity(mAdapter.intentForPosition(0, false));
            mPackageMonitor.unregister();
            mRegistered = false;
            finish();
            return;
        } else {
            setContentView(R.layout.resolver_list);

            final TextView empty = (TextView) findViewById(R.id.empty);
            empty.setVisibility(View.VISIBLE);

            mListView = (ListView) findViewById(R.id.resolver_list);
            mListView.setVisibility(View.GONE);
        }
        //其它初始化代码
        ..........
    }

```
<font color =red>*通过以上方法即可实现设置默认home的功能*</font>
#### 方案二【优势，可以通过长按home按键进行切换】
ActivityManagerService.java在每次启动时执行，每次都默认启动设定的launcher，当然，如果设定的launcher存在的话，设置其他的launcher为默认会无效，因为重新启动时setDefaultLauncher()会将当前默认launcher清除。只有在卸载了设定默认启动的launcher后才能设置其他launcher为默认启动.
修改AMS：
```
frameworks\base\services\java\com\android\server\am\ActivityManagerService.java
```
**新增客制化的method**
```java
private void setDefaultLauncher() {
        // get default component
        String packageName = "你指定的包名";//默认launcher包名
        String className = "你指定的类名";////默认launcher入口

        IPackageManager pm = ActivityThread.getPackageManager();

        //判断指定的launcher是否存在
        if(hasApkInstalled(packageName)) {

            Slog.i(TAG, "defautl packageName = " + packageName + ", default className = " + className);
            //清除当前默认launcher
            ArrayList<IntentFilter> intentList = new ArrayList<IntentFilter>();  
            ArrayList<ComponentName> cnList = new ArrayList<ComponentName>();  
            mContext.getPackageManager().getPreferredActivities(intentList, cnList, null);
            IntentFilter dhIF = null;  
            for(int i = 0; i < cnList.size(); i++) {  
                dhIF = intentList.get(i);  
                if(dhIF.hasAction(Intent.ACTION_MAIN) && dhIF.hasCategory(Intent.CATEGORY_HOME)) {
                    mContext.getPackageManager().clearPackagePreferredActivities(cnList.get(i).getPackageName());
                }
            }
            //获取所有launcher activity 
            Intent intent = new Intent(Intent.ACTION_MAIN);  
            intent.addCategory(Intent.CATEGORY_HOME);  
            List<ResolveInfo> list = new ArrayList<ResolveInfo>();  
            try {  
                list = pm.queryIntentActivities(intent, intent.resolveTypeIfNeeded(mContext.getContentResolver()), PackageManager.MATCH_DEFAULT_ONLY);  
            }catch (RemoteException e) {  
                throw new RuntimeException("Package manager has died", e);  
            }   
            // get all components and the best match  
            IntentFilter filter = new IntentFilter();  
            filter.addAction(Intent.ACTION_MAIN);  
            filter.addCategory(Intent.CATEGORY_HOME);  
            filter.addCategory(Intent.CATEGORY_DEFAULT);  
            final int N = list.size();
            //设置默认launcher 
            ComponentName launcher = new ComponentName(packageName, className);  
 
            ComponentName[] set = new ComponentName[N];  
            int defaultMatch = 0;  
            for (int i = 0; i < N; i++) {  
                ResolveInfo r = list.get(i);  
                set[i] = new ComponentName(r.activityInfo.packageName, r.activityInfo.name);  
                Slog.d(TAG, "r.activityInfo.packageName======= " + r.activityInfo.packageName);
                Slog.d(TAG, "r.activityInfo.name========= " + r.activityInfo.name);
                if(launcher.getClassName().equals(r.activityInfo.name)) {
                    defaultMatch = r.match;
                }
            }
            try {  
                pm.addPreferredActivity(filter, defaultMatch, set, launcher);
            } catch (RemoteException e) {  
                throw new RuntimeException("Package manager has died", e);  
            }   
             
             
        }//end if
         
    }  
 
    private static boolean hasApkInstalled(String pkgname) {
 
        try {
            mSelf.mContext.getPackageManager().getPackageInfo(pkgname,0);
        } catch(Exception e) {
            Slog.d(TAG, "PackageManager.NameNotFoundException: = " + e.getMessage());
            return false;
        }
        return true;
    }
```
然后在ActivityManagerService类中的
`boolean startHomeActivityLocked()`
方法第一行调用上面添加的
`setDefaultLauncher()`

```java
boolean startHomeActivityLocked() {
        if (mFactoryTest == SystemServer.FACTORY_TEST_LOW_LEVEL
                && mTopAction == null) {
            // We are running in factory test mode, but unable to find
            // the factory test app, so just sit around displaying the
            // error message and don't try to start anything.
            return false;
        }
 `
	//-------新增方法的执行位置---------------
        setDefaultLauncher();
         
        Intent intent = new Intent(
            mTopAction,
            mTopData != null ? Uri.parse(mTopData) : null);
        intent.setComponent(mTopComponent);
        if (mFactoryTest != SystemServer.FACTORY_TEST_LOW_LEVEL) {
            intent.addCategory(Intent.CATEGORY_HOME);
        }
        ActivityInfo aInfo =
            intent.resolveActivityInfo(mContext.getPackageManager(),
                    STOCK_PM_FLAGS);
        if (aInfo != null) {
            intent.setComponent(new ComponentName(
                    aInfo.applicationInfo.packageName, aInfo.name));
            // Don't do this if the home app is currently being
            // instrumented.
            ProcessRecord app = getProcessRecordLocked(aInfo.processName,
                    aInfo.applicationInfo.uid);
            if (app == null || app.instrumentationClass == null) {
                intent.setFlags(intent.getFlags() | Intent.FLAG_ACTIVITY_NEW_TASK);
                mMainStack.startActivityLocked(null, intent, null, null, 0, aInfo,
                        null, null, 0, 0, 0, false, false, null);
            }
        }
                  
        return true;
    }
```
  添加后的方法全部内容如上，重新编译android，烧录，开机就能够自动进入自定义的launcher
可以通过系统设置取消该launcher的默认设置，取消之后按home键会弹出launcher选择提示框
`frameworks\base\core\java\com\android\internal\app\ResolverActivity.java`
ResolverActivity类就是选择打开方式的弹出框
  
 顺便查看老外有没有类似的需求，在stackoverflow上面一查询还真发现有一些精彩的答案（O(∩_∩)O哈哈~，看来奸商全球通用）：
 [How to set default app launcher programmatically?](http://stackoverflow.com/questions/27991656/how-to-set-default-app-launcher-programmatically) 
 [How to reset default launcher/home screen replacement?](http://stackoverflow.com/questions/15537522/how-to-reset-default-launcher-home-screen-replacement) 
