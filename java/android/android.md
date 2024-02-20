# Android



创建一个`empty view project`

修改gradle相关文件下载地址：

修改`gradle-wrapper.properties`文件中的`distributionUrl`属性,改成国内镜像网站

`https://mirrors.cloud.tencent.com/gradle/gradle-8.0-bin.zip`

下载完配置后打开

`res/layout/activity_main.xml`，就可以对布局进行设置



创建一个虚拟设置手机来启动app

![image-20231127203314726](image\image-20231127203314726.png)



### TextView 控件

```java
		TextView textView = findViewById(R.id.tv_one);
        textView.setText("会覆盖原有的数值");
```

组件属性的设置在正规开发中要写在`res/values`中

调用的话`@标签类型/标签设置名字`



#### 带阴影的 TextView

![image-20231127211222192](image\JWT.md)

#### 跑马灯效果的 TextView



```xml
<!-- layout_width可以两种枚举类型，一种是匹配父容器，另一种是视图的大小应仅足以包含其内容（加上填充），可以设置具体数值200dp-->
    <!-- id 属性用来标识组件，“@+id/tv_one”，可以在启动器中获取到这个组件-->
    <!-- text 文本内容-->
    <!-- textColor 文本颜色:有八位，前两位表示透明度，后六位是三原色 -->
    <!-- textSize 字体大小，单位是sp-->
    <!-- backgroud控件背景，可以是颜色，也可以是图片-->
    <!-- gravity 组件布局模式-->
    <!-- shadowColor阴影颜色，shadowRadius阴影的模糊程度，shadowDx阴影在水平方向的偏移，shadowDy。。。-->
    <!-- singleLine内容单行显示、focusable是否可获取焦点，
    focusableInTouchMode用于控制视图字啊触摸模式下是否可以聚焦、
    ellipsize在哪里省略文本、marqueeRepeatLimit字幕动画重复次数
    自定义类重写isFocused()方法、增加点击事件、<requestFocus/>请求聚焦
    -->
```



### 选择器 selector

在根目录`new module`创建新的项目

在`res/drawabel`目录下可以导入绘制图片，然后通过设置好的图片选择器来引用。

也可以自定义图片选择器xml文件，可以设置参数达到不同的效果。

```xml
<selector>
    <item>...</item>
</selector>
```

![image-20231128134425876](image\image-20231128134425876.png)

然后控件利用`@drawable/{filename}`来引用绘制图片绘制自定义图片。

可以在`/res`目录下创建color文件夹，在里面定义颜色选择器



### Button 控件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
>
    <Button
        android:id="@+id/btn"
        android:layout_width="200dp"
        android:layout_height="100dp"/>


</LinearLayout>
```

#### 事件处理

```java
		static final  String TAG = "leo";		
....

		Button btn = findViewById(R.id.btn);
        // 点击事件
        btn.setOnClickListener(new View.OnClickListener(){
            @Override
            public void onClick(View v){

            }
        });
        // 长按事件
		// 返回值为true时，不会执行onClick
        btn.setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View view) {
                return false;
            }
        });
        // 触摸事件
		// 触摸事件有三种，点击，在上面移动，松开
		// TAG是给是给日志目录设置路径
		// 返回值为true时，只执行onToch
        btn.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                Log.e(TAG, "onTouch: " + motionEvent.getAction());
                return false;
            }
        });
```



### EdiText

继承了TextView

![image-20231128140219643](image\image-20231128140219643.png)

### ImageView

![image-20231128140909753](image\image-20231128140909753.png)

**缩放类型**:![image-20231128140946218](image\image-20231128140946218.png)

### ProgressBar 进度条

![image-20231128142357307](image\image-20231128142357307.png)



### Notification与NotigicationManager

![image-20231128142756103](image\image-20231128142756103.png)

![image-20231128143015073](image\image-20231128143015073.png)

![image-20231128143542443](image\image-20231128143542443.png)

```java
notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);

        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel = new NotificationChannel("leo", "测试通知",
                    NotificationManager.IMPORTANCE_HIGH);
            notificationManager.createNotificationChannel(channel);
        }
        Notification notification = new NotificationCompat.Builder(this,"leo")
                .setContentTitle("官方通知")
                .setContentText("测试内容")
                .setSmallIcon(R.drawable.baseline_add_ic_call_24)
                .build();
```



### ToolBar



### AlertDialog

![image-20231128145320929](image\image-20231128145320929.png)

### PopupWindow

![image-20231128145402978](image\image-20231128145402978.png)

## 布局

### LinearLayout

![image-20231128145642447](image\image-20231128145642447.png)

### RelativeLayout



![image-20231128145841541](image\image-20231128145841541.png)



![image-20231128150001023](image\image-20231128150001023.png)

### FrameLayout

> 类似叠层，不断在上面布局，子布局的覆盖父布局



### TableLayout

控件直接占用一行

可以用TableRow来将多个组件写入一行

![image-20231128150416764](image\image-20231128150416764.png)



### GridLayout

网格布局

![image-20231128150631050](image\image-20231128150631050.png)



![image-20231128150704078](image\image-20231128150704078.png)



### ConstraintLayout

约束布局

一般在UI界面进行设计





## 组件

### ListView

创建一个列表内容组件`list_item.xml`

将相关数据类放入list容器中

创建自定义适配器，然后继承 `BaseAdapter`

```java
public class MyAdapter extends BaseAdapter {

    private List<Object> data;
    private Context context;

    public MyAdapter(List<Object> data, Context context) {
        this.data = data;
        this.context = context;
    }

    @Override
    public int getCount() {
        return data.size();
    }

    @Override
    public Object getItem(int i) {
        return null;
    }

    @Override
    public long getItemId(int i) {
        return i;
    }

    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        ViewHolder viewHolder;
        if(view == null){
            viewHolder = new ViewHolder();
            view = LayoutInflater.from(context).inflate(R.layout.list_item,viewGroup,false);
            viewHolder.textView = view.findViewById(R.id.tv);
            view.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder)view.getTag();
        }
        viewHolder.textView.setText(data.get(i).toString());
        return view;
    }
    private final class ViewHolder{
        TextView textView;
    }
}
```



### ViewPage

...



### Fragment

* 具备声明周期，可以看成一个小的activity

  

* 必须委托在activity中才能运行



创建一个空白的fragment类

还要在activity中加入fragment

#### 动态添加 Fragment

![image-20231128202345451](image\image-20231128202345451.png)

#### Fragment 之间的 activity的通信

demo02

* activity与fragment的通信

  原生方案：Bundle

  java 语言中类与类自己通信常用方案：接口（接口弱连接）

  其它方案：eventBus、LiveData（发布订阅模式）

  

#### Fragment + viewPage 滑动



### ViewPager



