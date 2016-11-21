Title: Android辅助功能AccessibilityService的使用
Date: 2016-11-17
Modified: 2016-11-17
Tags: Android,AccessibilityService
Slug: android_accessibilityService_apply
Authors: Cokernut
Summary: Android辅助功能AccessibilityService的使用

## AccessibilityService官方简介：  
> The classes in this package are used for development of accessibility service that provide alternative or augmented feedback to the user.  
使用这个类可以开发用于给用户提供替换或者是增强反馈的辅助功能服务。  
-----
> An AccessibilityService runs in the background and receives callbacks by the system when AccessibilityEvents are fired. Such events denote some state transition in the user interface, for example, the focus has changed, a button has been clicked, etc. Such a service can optionally request the capability for querying the content of the active window. Development of an accessibility service requires extends this class and implements its abstract methods.  
一个AccessibilityService在后台运行并接收系统AccessibilityEvents事件的回调，当用户界面的状态发生改变时会触发AccessibilityEvents事件，例如焦点的变化，点击一个按钮。
这个服务可以获取到活动窗口的内容，开发一个辅助功能服务需要继承AccessibilityService并实现其中的抽象方法。  
-----
> An AccessibilityServiceInfo describes an AccessibilityServiceInfo. The system notifies an AccessibilityService for AccessibilityEvents according to the information encapsulated in this class.  
一个AccessibilityService有一个用于描述AccessibilityService的AccessibilityServiceInfo对象，系统会通知AccessibilityService根据AccessibilityServiceInfo把信息装进AccessibilityEvents中。

## 继承AccessibilityService并实现其中的抽象方法。
> 下面是我Service类：
```java
public class MyService extends AccessibilityService {
    private int code = INSTALL;
    private static final int INSTALL = 0;
    private static final int NEXT = 1;
    private static final int FINISH = 2;
    /**
     * 页面变化回调事件
     * @param event event.getEventType() 当前事件的类型;
     *              event.getClassName() 当前类的名称;
     *              event.getSource() 当前页面中的节点信息；
     *              event.getPackageName() 事件源所在的包名
     */
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
        // 事件页面节点信息不为空
        if (event.getSource() != null) {
            // 判断事件页面所在的包名，这里是自己
            if (event.getPackageName().equals(getApplicationContext().getPackageName())) {
                switch (code) {
                    case INSTALL:
                        click(event, "安装", TextView.class.getName());
                        Log.d("test=======", "安装");
                        code = NEXT;
                        break;
                    case NEXT:
                        click(event, "下一步", Button.class.getName());
                        Log.d("test=======", "下一步");
                        code = FINISH;
                        break;
                    case FINISH:
                        click(event, "完成", TextView.class.getName());
                        Log.d("test=======", "完成");
                        code = INSTALL;
                        break;
                    default:
                        break;
                }
            }
        } else {
            Log.d("test=====", "the source = null");
        }
    }

    /**
     * 模拟点击
     * @param event 事件
     * @param text 按钮文字
     * @param widgetType 按钮类型，如android.widget.Button，android.widget.TextView
     */
    private void click(AccessibilityEvent event, String text, String widgetType) {
        // 事件页面节点信息不为空
        if (event.getSource() != null) {
            // 根据Text搜索所有符合条件的节点, 模糊搜索方式; 还可以通过ID来精确搜索findAccessibilityNodeInfosByViewId
            List<AccessibilityNodeInfo> stop_nodes = event.getSource().findAccessibilityNodeInfosByText(text);
            // 遍历节点
            if (stop_nodes != null && !stop_nodes.isEmpty()) {
                AccessibilityNodeInfo node;
                for (int i = 0; i < stop_nodes.size(); i++) {
                    node = stop_nodes.get(i);
                    // 判断按钮类型
                    if (node.getClassName().equals(widgetType)) {
                        // 可用则模拟点击
                        if (node.isEnabled()) {
                            node.performAction(AccessibilityNodeInfo.ACTION_CLICK);
                        }
                    }
                }
            }
        }
    }

    /**
     * 中断AccessibilityService的反馈时调用
     */
    @Override
    public void onInterrupt() {
        Log.d("test=====", "Interrupt");
    }
}

```

> AccessibilityService里几个重要的方法：  

+ onServiceConnected() - 可选。系统会在成功连接上你的服务的时候调用这个方法，在这个方法里你可以做一下初始化工作，例如设备的声音震动管理，也可以调用setServiceInfo()进行配置AccessibilityServiceInfo。
+ onAccessibilityEvent() - 必须。通过这个函数可以接收系统发送来的AccessibilityEvent，接收来的AccessibilityEvent是经过过滤的，过滤是在配置工作时设置的。
+ onInterrupt() - 必须。这个在系统想要中断AccessibilityService返给的响应时会调用。在整个生命周期里会被调用多次。
+ onUnbind() - 可选。在系统将要关闭这个AccessibilityService会被调用。在这个方法中进行一些释放资源的工作。

> 之后在AndroidManifest文件里注册并添加相应的权限：
```xml
<service
    android:name=".MyService"
    android:label="辅助功能"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
    <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService" />
    </intent-filter>
    ...
</service>
```

## 配置AccessibilityService
1. 可以在onServiceConnected()方法里进行，建立一个AccessibilityServiceInfo对象，通过这个对象设置监听系统事件类型，服务的反馈类型（震动，语音，声音），事件时间间隔，你想要监听的App的包名。最后调用setServiceInfo()进行设置，如：
```java
 @Override
    protected void onServiceConnected() {
        super.onServiceConnected();
        AccessibilityServiceInfo info = new AccessibilityServiceInfo();
        info.eventTypes = AccessibilityEvent.TYPES_ALL_MASK;
        info.packageNames = PACKAGE_NAMES; 
        ...配置
        setServiceInfo(info);
    }
```
2. 从Android4.0开始，开发者可以通过在AndroidManifest里添加<meta-data>标签配置AccessibilityService，在标签里指出配置文件的位置，如：  
> res/xml/accessibility_service_info.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/accessibility_service_description"
    android:accessibilityEventTypes="typeAllMask"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:notificationTimeout="100"
    android:accessibilityFlags="flagDefault"
    android:canRetrieveWindowContent="true"
    android:packageNames="top.cokernut.sample"
    android:settingsActivity="com.example.android.accessibility.ServiceSettingsActivity"  />

    <!--
         android:description="@string/accessibility_service_description" 描述
         android:accessibilityEventTypes="typeAllMask"  事件类型
         android:accessibilityFeedbackType="feedbackGeneric" 反馈类型，声音、震动等
         android:canRetrieveWindowContent="true", 允许获取手机页面中的信息
         android:packageNames="top.cokernut.sample" 要监听的包名，过滤作用
         android:settingsActivity="packname.android.accessibility.ServiceSettingsActivity" packname写自己App的包名
    -->
```
> 事件类型（EventType）：
```
        #TYPES_ALL_MASK：所有类型
        #TYPE_VIEW_CLICKED ：单击
        #TYPE_VIEW_LONG_CLICKED ：长按
        #TYPE_VIEW_SELECTED ：选中
        #TYPE_VIEW_FOCUSED ：获取焦点
        #TYPE_VIEW_TEXT_CHANGED ：文字改变
        #TYPE_WINDOW_STATE_CHANGED ：窗口状态改变
        #TYPE_NOTIFICATION_STATE_CHANGED ：通知状态改变
        #TYPE_VIEW_HOVER_ENTER
        #TYPE_VIEW_HOVER_EXIT
        #TYPE_TOUCH_EXPLORATION_GESTURE_START
        #TYPE_TOUCH_EXPLORATION_GESTURE_END
        #TYPE_WINDOW_CONTENT_CHANGED
        #TYPE_VIEW_SCROLLED
        #TYPE_VIEW_TEXT_SELECTION_CHANGED
        #TYPE_ANNOUNCEMENT
        #TYPE_VIEW_TEXT_TRAVERSED_AT_MOVEMENT_GRANULARITY
        #TYPE_GESTURE_DETECTION_START
        #TYPE_GESTURE_DETECTION_END
        #TYPE_TOUCH_INTERACTION_START
        #TYPE_TOUCH_INTERACTION_END
        #TYPE_WINDOWS_CHANGED
```
> 然后在AndroidManifest文件里把配置文件配置到AccessibilityService上：
``` xml
<service
    android:name=".MyService"
    android:label="辅助功能"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
    <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService" />
    </intent-filter>

    <meta-data
        android:name="android.accessibilityservice"
        android:resource="@xml/accessibility_service_info" />
</service>
```
到此一个AccessibilityService的开发就完成了。

## 其他
> MainActivity：
```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private TextView mInstallTV;
    private TextView mFinishTV;
    private TextView mStartTV;
    private TextView mStopTV;

    private Button mInstallBT;
    private Button mNextBT;

    private int num = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mInstallTV      = (TextView) findViewById(R.id.tv_install);
        mStartTV        = (TextView) findViewById(R.id.tv_start);
        mStopTV         = (TextView) findViewById(R.id.tv_stop);
        mFinishTV       = (TextView) findViewById(R.id.tv_finish);
        mInstallBT      = (Button) findViewById(R.id.bt_install);
        mNextBT         = (Button) findViewById(R.id.bt_next);
        mInstallTV      .setOnClickListener(this);
        mFinishTV       .setOnClickListener(this);
        mInstallBT      .setOnClickListener(this);
        mNextBT         .setOnClickListener(this);
        mStartTV        .setOnClickListener(this);
        mStopTV         .setOnClickListener(this);
    }

    @Override
    protected void onRestart() {
        super.onRestart();
        num = 0;
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.tv_start:
                if (serviceIsRunning()) {
                    showToast("服务已经在运行！");
                } else {
                    startAccessibilityService();
                }
                break;
            case R.id.tv_stop:
                startActivity(new Intent(MainActivity.this, SecondActivity.class));
                break;
            case R.id.tv_install:
                showToast(((TextView)v).getText().toString());
                break;
            case R.id.tv_finish:
                showToast(((TextView)v).getText().toString());
                num++;
                if (num > 5) {
                    startActivity(new Intent("top.cokernut.sample.SecondActivity"));
                }
                break;
            case R.id.bt_install:
                showToast(((Button)v).getText().toString());
                break;
            case R.id.bt_next:
                showToast(((Button)v).getText().toString());
                break;
            default:
                showToast("未知按钮");
                break;
        }
    }

    private void showToast(final String text) {
        //AccessibilityService触发事件是异步的，要回到UI线程改变UI
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainActivity.this, "点击了" + text, Toast.LENGTH_LONG).show();
                Log.d("text=====", num + "=====" + text);
            }
        });
    }

    /**
     * 判断自己的应用的AccessibilityService是否在运行
     *
     * @return
     */
    private boolean serviceIsRunning() {
        ActivityManager am = (ActivityManager) getApplicationContext().getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.RunningServiceInfo> services = am.getRunningServices(Short.MAX_VALUE);
        for (ActivityManager.RunningServiceInfo info : services) {
            if (info.service.getClassName().equals(getPackageName() + ".MyService")) {
                return true;
            }
        }
        return false;
    }

    /**
     * 前往设置界面开启服务
     */
    private void startAccessibilityService() {
        new AlertDialog.Builder(this)
                .setTitle("开启辅助功能")
                .setIcon(R.mipmap.ic_launcher)
                .setMessage("使用此项功能需要您开启辅助功能")
                .setPositiveButton("立即开启", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                // 隐式调用系统设置界面
                                Intent intent = new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
                                startActivity(intent);
                            }
                        }).create().show();
    }
}
```
> activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="top.cokernut.sample.MainActivity">

    <LinearLayout
        android:id="@+id/activity_main"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:gravity="center_horizontal"
        android:paddingBottom="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin">

        <TextView
            android:id="@+id/tv_install"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="30dp"
            android:background="#00AA66"
            android:layout_margin="10dp"
            android:text="安装TV" />

        <Button
            android:id="@+id/bt_install"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="30dp"
            android:text="安装Btn" />

        <Button
            android:id="@+id/bt_next"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="30dp"
            android:text="下一步Btn" />

        <TextView
            android:id="@+id/tv_finish"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="30dp"
            android:background="#00AA66"
            android:layout_margin="10dp"
            android:text="完成" />

        <TextView
            android:id="@+id/tv_start"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="30dp"
            android:background="#0088FF"
            android:layout_margin="10dp"
            android:text="开始" />
        <TextView
            android:id="@+id/tv_stop"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="30dp"
            android:background="#0088FF"
            android:layout_margin="10dp"
            android:text="停止" />
    </LinearLayout>
</ScrollView>
```
> SecondActivity就一个TextView显示信息，无操作就不贴了。

<font size=5>[源代码:]("https://github.com/cokernut/AccessibilityServiceSample") <https://github.com/cokernut/AccessibilityServiceSample></font>