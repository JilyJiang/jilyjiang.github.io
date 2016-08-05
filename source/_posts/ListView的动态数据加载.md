layout: post
title: 译：ListView的动态数据加载
comment: true
tags: [Android, ListView, 技术]
date: 2016-08-04 22:11:14
updated: 2016-08-04 22:11:14
---

------
在很多情况下，ListView需要显示数据集合，在运行时加载的动态数据。 考虑到一些你可能使用的应用程序，新闻阅读应用程序，社交媒体，电视节目指南等等。 所有这些应用都将在运行时从服务器加载数据。 通常情况下，应用程序会显示在列表这些数据。 要做到这一点，通常会使用REST Web服务，其中请求使用HTTP消息发和响应返回序列化对象，常用于XML或JSON格式。 让我们来看一个例子，让我们可以探索其中的一些概念。

## 通过使用Picasa的网络数据动态更新ListView
在这个例子中，我们将构建一个从Picasa网络API加载数据并显示在列表中一个ListView。 我们将在几个步骤做到这一点。

* 首先，我们将创建一个空的数据集一个ListView。
* 其次，对于填充的数据将被检索。
* 最后，在需要时，我们将加载为每个项目中的图像列表
要做到这一点，我们将利用异步任务，使请求的Web API，这样我们就不会阻塞主UI线程。 我们将要求把数据返回为一个JSON字符串，JSON的解码库比较容易，易于使用GSON库中的JSON。 我们来看一下如何编写此之前，让我们来看看参与创建这种类型的ListView的步骤。

## 初始化的ListView

在我们可以开始加载数据为ListView，我们首先需要将其设置在初始空状态。 要做到这一点，我们将使用ListActivity将加载布局列表。 布局将包含一个列表视图和一个空的view。 然后，ListActivity将创建一个适配器，是空的，并将其分配给ListView。 由于适配器是空的，并且不包含任何数据，在ListView将显示空视图，在这种情况下，从布局装入一个TextView。 图1概括初始化过程。

初始化一个空的ListView
![fig01](http://oa1wnpe3m.bkt.clouddn.com/fig01.png) 
图1：初始化一个空的ListView

## 通过AsyncTask动态加载ListView数据

一旦我们初始化ListView控件，然后我们需要检索的数据来填充它。 要做到这一点，ListActivty启动异步任务从Picasa服务时加载数据。 这里，我们通过一个HTTP使用Picasa API的请求得到。 一旦我们获取数据从Picasa回来，我们更新了ListAdapter，以便它现在有这个数据的填充ArrayList中，我们再调用notifyDataSetChanged使ListView控件可以更新并与填充的数据绘制。 图2给出了该方法的用于从所述的Picasa服务检索数据的概述。

与异步任务加载Picasa的数据
![fig02](http://oa1wnpe3m.bkt.clouddn.com/fig02.png) 
图2加载Picasa的数据

## 加载图像的位图

我们刚刚加载并填充到ArrayList中的数据中包含的信息，如标题，描述和URL路径的图像。 它不包含对图像本身的实际位图数据。 因此，需要做的最后一件事是加载图像数据在每个列表项的图像。 如已经提到的，该图像数据被延迟加载，并且，直到显示与图像的列表项，对于该图像数据不会被检索。 因此，如何只加载图像数据时，我们需要它？

每个Adapter有一个getView方法被调用时，列表需要显示列表项。 因此，当你通过一个列表中滚动， getView将每次显示一个新的视图时调用。 正是在这里getView ，我们实现了图像数据的延迟加载。 请记住，我们已经填充进数据的ArrayList。 所以，每次getView被调用时，我们简单地分配的标题和描述中的列表项的相应意见文本字符串。

对于每一个图片，我们需要做一点额外的工作。 填充数据是使用我们提供了图像的URL地址。 我们使用这个网址进行到Picasa中服务器的另一个异步调用从适配器中getView方法。 我们将用它来 ​​使这个异步调用的类是ImageDownloader 。 一旦数据被加载，我们更新用于各列表项目的ImageView的。 加入少许UI动画效果，我们还可以使用默认的图像和新加载的图像之间的交叉淡入淡出的效果。

图3描述了这一过程。
![fig03](http://oa1wnpe3m.bkt.clouddn.com/fig03.png) 
加载图像上的列表视图调用getView每个列表项

图3装入图像上的列表视图每个项目调用getView

从本质上讲，有三个步骤，我们的例子：
* 第一，我们初始化一个空的ListView; 
* 第二，我们加载填充的数据; 并且以完成，如显示在列表中每个项目我们然后慢慢地加载的图像数据。

## 实现一个动态的ListView
随着我们如何实现我们的ListView在我们的脑海中记忆犹新的概述，让我们来看看我们如何真正去了解动态数据加载到一个ListView的实施细节。 我们必须做的第一件事就是为我们的列表视图布局XML，如清单1所示。

清单1包含ListView控件layout
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
       xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:gravity="center"
       android:orientation="vertical" >

       <ListView
          android:id="@android:id/list"
          android:fadingEdge="vertical"
          android:fadingEdgeLength="10dp"
          android:longClickable="true"
          android:listSelector="@drawable/list_selector_background"
          android:layout_width="match_parent"
          android:layout_height="match_parent" >
       </ListView>

       <TextView
          android:id="@android:id/empty"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_gravity="center"
          android:text="Loading feed data..." />

</LinearLayout>
```
1 Listview with id of "list"
2 TextView with id of "empty
在这里，我们有一个包含view的LinearLayout中，一个ListView（＃1）和一个TextView（＃2）。 ListView中有显示数据的有关TextView的名单，我们将加载，但什么？ 这种观点只显示，当我们没有数据，但现在我们只是要使用显示字符串一个TextView任何类型的视图“加载进数据。”最重要的一点到这里是通知这两个ListView和TextView的有特定的ID必须是正确的。 ListView控件必须有@android的ID：id/list和TextView中必须有和@android的ID:id/empty。 否则，ListActivty将不知道使用哪种意见的清单及其空另一种观点时，有没有数据显示。

所以这是完成的主要版面，让我们不要忘记，我们还需要为每个列表项的布局。 对于这一点，我们就重用我们在前面的例子中定义这里要再次重申在上市2列表项的布局。

清单2 listview子item的view布局文件
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
       xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="wrap_content" >

       <ImageView
          android:id="@+id/listImage"
          android:layout_width="100dp"
          android:layout_height="100dp"
          android:layout_alignParentLeft="true"
          android:layout_centerVertical="true"
          android:layout_margin="10dp"
          android:background="#ffcccccc" />

       <TextView
          android:id="@+id/listTitle"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_alignTop="@+id/listImage"
          android:layout_toRightOf="@+id/listImage"
          android:text="A List item title"
          android:textSize="16sp"
          android:textStyle="bold" />

       <TextView
          android:id="@+id/listDescription"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_below="@+id/listTitle"
          android:layout_marginTop="5dp"
          android:maxLines="4"
          android:layout_toRightOf="@+id/listImage"
          android:text="The List item description"
          android:textSize="14sp" />

</RelativeLayout>
```

在定义的布局，我们现在需要一个ListActivity。 清单3显示了我们ListActivity称为DynamicListviewActivity。 这仍然是一个非常简单的活动。

清单3 ListActivity为动态的ListView

```java
public class DynamicListViewActivity extends ListActivity {

       public void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.list);                              //1

          ImageListAdapter adapter = new ImageListAdapter(this);
          setListAdapter(adapter);                                    //2

          LoadFeedData loadFeedData = new LoadFeedData(adapter);
          loadFeedData.execute();                                     //3
       }
}
```
在XML定义
	1加载列表布局
	2创建并为ListView设置适配器。 最初，这适配器将是空的，没有包含数据。
	3实例化和执行LoadFeedData，在后台异步加载的Picasa数据。 
本次活动的适配器确实比我们前面例子中的适配器多一点，但是，现在，让我们来看看LoadFeedData ，我们实例化类并调用在执行上的＃3 onCreate我们ListActivity的方法。 该LoadFeedData类希望有一个相当不言自明的名称。 它的任务是加载进数据; 在这个例子中，它会检索来自Picasa网络API，它的数据。 让我们来看看这个类。

清单用于装载的饲料数据4 LoadFeedData类

```java
public class LoadFeedData extends
          AsyncTask> {                           #1

       private final String mUrl =                               #2
          "http://picasaweb.google.com/data/feed/api/all?kind=photo&q=" +
          "sunset%20landscape&alt=json&max-results=20&thumbsize=144c";

       private final ImageListAdapter mAdapter;

       public LoadFeedData(ImageListAdapter adapter) {           #3
          mAdapter = adapter;
       }

       private InputStream retrieveStream(String url) {
          DefaultHttpClient client = new DefaultHttpClient();
          HttpGet httpGet = null;
          httpGet = new HttpGet(url);

          HttpResponse httpResponse = null;
          try {
             httpResponse = client.execute(httpGet);
             HttpEntity getResponseEntity = httpResponse.getEntity();
             return getResponseEntity.getContent();
          } catch (IOException e) {
             httpGet.abort();
          }
          return null;
       }

       @Override
       protected ArrayList<Entry> doInBackground(Void... params) {
          InputStream source = retrieveStream(mUrl);                   #4
          Reader reader = null;
          try {
             reader = new InputStreamReader(source);
          } catch (Exception e) {
             return null;
          }
          Gson gson = new Gson();
          SearchResult result = gson.fromJson(reader,SearchResult.class);      #5
          return result.getFeed().getEntry();
       }

       protected void onPostExecute(ArrayList<Entry> entries) {
          mAdapter.upDateEntries(entries);                        #6
       }
}
``` 
 ＃1 LoadFeedData延伸的AsyncTask
 ＃2设置URL，这构成了一个查询到的Picasa API
 ＃3 LoadFeedData构造
 ＃4 retrieveStream方法调用返回的InputStream到Picasa数据
 ＃5呼叫fromJson方式转变JSON字符串转换为对象
在适配器＃6呼叫UpdateEntries数据的检索数组来填充它 
在LoadFeedData左看右看，我们可以看到，这是一个异步类。 Android的异步类允许我们在不同的线程主的Andr​​oid UI线程执行代码。 通过调用一个异步任务执行方法，我们指示它执行的是在一个单独的后台线程其doInBackground方法的任何代码。 在异步任务的完整解释超出了这个例子的范围，但是对这些非常有用的类的更多信息可以在Android开发者网站找到[AsyncTask](https://developer.android.com/training/improving-layouts/smooth-scrolling.html)
通过本类的​​代码步进，我们可以看到，我们做的第一件事就是建立一个URL（＃2）。 此URL拼成一个查询到的Picasa API。 该参数指出，我们要检索最多20幅影像的缩略图大小的，并涉及到景观和日落。 此外，我们声明，我们希望数据以JSON格式返回。 用于保持简单起见这个URL被硬编码，但是当然有可能建立的网址，可以在运行时做出许多不同类型的查询的目的。 LoadFeedData类的构造函数取到适配器的参考为ListView（＃3），这样我们就可以通知它，当我们从Picasa中API检索提要数据。

所有这些类的主要工作发生在doInBackground方法。 这里我们使用的私有方法retrieveStream（＃4）返回一个InputStream。 InputStream中包含一个JSON格式的序列化的数据对象。 为了将JSON字符串，我们可以一起工作的对象，我们使用一个类从GSON库并调用该方法fromJson（＃5），而传递的InputStream，我们希望映射到JSON数据的类。

要分析我们使用GSON JSON数据，这是图书馆。 该GSON库反序列化从现有的JSON字符串数据的强大方式，也可以用于序列化对象JSON字符串。 在GSON的更多信息可以在[这里找到](https://sites.google.com/site/gson/Home)。

我们所要做的，从JSON字符串得到的数据是供应有直接映射到JSON字符串的属性的属性类。 该GSON类会照顾细节我们。

一旦我们从Picasa网络API检索到的数据，我们再通过这个数据作为输入数组通过UpdateEntries方法反对适配器（＃6）。 然后在适配器将具有图像数据的填充ArrayList和还通知在ListView，这是使得在ListView可显示该数据的情况。 让我们来看看，我们在这个例子中使用的适配器一探究竟。

清单5 ImageListAdapter用于填充与数据和图像的ListView

```java
public class ImageListAdapter extends BaseAdapter {

       private Context mContext;

       private LayoutInflater mLayoutInflater;                              #1

       private ArrayList<Entry> mEntries = new ArrayList<Entry>();          #2

       private final ImageDownloader mImageDownloader;                      #3

       public ImageListAdapter(Context context) {                           #4
          mContext = context;
          mLayoutInflater = (LayoutInflater) mContext
                   .getSystemService(Context.LAYOUT_INFLATER_SERVICE);
          mImageDownloader = new ImageDownloader(context);
       }

       @Override
       public int getCount() {
          return mEntries.size();
       }

       @Override
       public Object getItem(int position) {
          return mEntries.get(position);
       }

       @Override
       public long getItemId(int position) {
          return position;
       }

       @Override
       public View getView(int position, View convertView,
             ViewGroup parent) {                                           #5
          RelativeLayout itemView;
          if (convertView == null) {                                        #6
             itemView = (RelativeLayout) mLayoutInflater.inflate(
                      R.layout.list_item, parent, false);

          } else {
             itemView = (RelativeLayout) convertView;
          }

          ImageView imageView = (ImageView)
             itemView.findViewById(R.id.listImage);                        #7
          TextView titleText = (TextView)
             itemView.findViewById(R.id.listTitle);                        #7
          TextView descriptionText = (TextView)
             itemView.findViewById(R.id.listDescription);                  #7

          String imageUrl = mEntries.get(position).getContent().getSrc();   #8
          mImageDownloader.download(imageUrl, imageView);                   #9

          String title = mEntries.get(position).getTitle().getTitle();
          titleText.setText(title);
          String description =
             mEntries.get(position).getSummary().getDescription;
          if (description.trim().length() == 0) {
             description = "Sorry, no description for this image.";
          }
          descriptionText.setText(description);

          return itemView;
       }

       public void upDateEntries(ArrayList<Entry> entries) {
          mEntries = entries;
          notifyDataSetChanged();
       }
}
```
 
 ＃1用于充气列表项布局的布局气筒
 ＃2项类从Picasa服务时，存储数据，该数组最初为空，但将被LoadDataFeed异步任务来填充
 ＃3 ImageDownloader类。 用于延迟加载图像列表的每个项目的另一个异步任务。
 ＃4的构造ImageListAdapter。 设置引用的背景下，建立布局充气和ImageDownloader。
适配器＃5 getView方法，这种方法被称为每ListView的需要显示一个列表项的时间。
 ＃6检查covertView为null，看看我们是否可以重用ListView控件传递一个未使用的视图。
 ＃7获取到图像，标题和描述引用列表项目，这样我们就可以从Picasa服务时，数据填充。
 ＃8获取来自Picasa中的数据列表项图像的URL。
上ImageDownloader＃9呼叫下载方法来启动一个异步的任务，将下载的图像数据，并更新图像的列表项。 
在这里，我们可以看到，我们的适配器被称为ImageListAdapter。 该适配器与所有我们所期望的方法相当典型的适配器。 当然，getView（＃5）为，其中大部分的工作就完成了。

当getView叫什么名字？ 
请记住，适配器的getView方法被调用每一个列表需要显示列表项的时间。 这样，当最初显示一列表项，此方法将被称作多次，使得当前显示可以与包含在适配器中的数据来填充每个列表项。 当我们滚动列表和新列表项映入眼帘，getView将再次为显示各列表项调用。

我们的getView方法是一个相当标准的例子。 这种方法的整体目标是用数据填充每个列表项，在这种情况下shelves，我们从Picasa网络API检索ArrayList中。

有一件事是不同的这里所做的是我们如何填充列表项图像。 在这里，我们得到的URL（＃8），用于此列表项图像，并同时通过网址并向ImageView的列表项的参考ImageDownloader通过调用其下载方法类（＃9）。 此方法将异步加载数据，图像，一旦它完成更新图像。 这是在一个懒惰的方式进行，因为getView当列表项需要显示将只被调用，因此，只有当一个列表项所示的图像只被检索并在需要的图像。

的的主要目的ImageDownloader是异步加载图像，但它在内存中还缓存图像。 我们ImageDownloader类有向它提出的，所以我们可以添加缓存图像到磁盘的可能性，也因此我们可以添加我们展示列表中的初始图像和最终下载的图像之间的过渡交叉淡入淡出了一些调整。

要添加淡入淡出功能，我们使用一个称为另一个类CrossFadeDrawable 。 这个类也是基于以前写的代码。 罗曼盖伊，一个谷歌工程师，最初创建这个类叫做shelves的一个开源项目，它在它的一些漂亮的代码技术，值得花时间去看看。 我们已经取得了一个调整这一类，因此它正是我们所需要的。 在这里，我们的好办法是添加扩展我们下载的镜像我们开始图像的大小矩阵转换，因为原来的类假定默认启动图像在大小与下载的终端形象一样。

如果您想了解更多关于ImageDownloader和CrossFadeDrawable类，从github的网址上区查找相关知识点

## 总结
 我们涵盖了所有我们需要加载动态数据和图像，并在列表中显示出来，每个之间的过渡淡入淡出不同的技术列表项默认的形象和它的最终下载的图像。
## 参考链接：
>[Android ListViews with Dynamic Data](http://www.developerfusion.com/article/145373/android-listviews-with-dynamic-data/)
