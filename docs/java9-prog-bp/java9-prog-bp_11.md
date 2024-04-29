# 第十一章：DeskDroid-用于 Android 手机的桌面客户端

我们终于来到了我们的最终项目。为了结束我们在这里的时光，我们将构建一个非常实用的应用程序，让我们可以在桌面上轻松发送和接收短信。现在市场上有许多产品可以做到这一点，但它们通常需要第三方服务，这意味着您的消息会通过其他人的服务器传输。对于注重隐私的人来说，这可能是一个真正的问题。我们将构建一个 100%本地的系统。

构建应用程序将涵盖几个不同的主题，有些熟悉，有些新的。该列表包括以下内容：

+   Android 应用程序

+   Android 服务

+   REST 服务器

+   服务器发送事件以进行事件/数据流式传输

+   使用内容提供程序访问数据

在我们一起度过美好时光的过程中，还会有许多其他小细节。

# 入门

这个项目将有两个部分：

+   Android 应用程序/服务器（当然不要与应用服务器混淆）

+   桌面/JavaFX 应用程序

桌面部分没有服务器部分就有点无用，所以我们将首先构建 Android 端。

# 创建 Android 项目

虽然到目前为止我们大部分工作都在使用 NetBeans，但我们将再次使用 Android Studio 进行项目的这一部分。虽然 NetBeans 对 Android 有一定程度的支持，但截至目前，该项目似乎已经停滞不前。另一方面，Android Studio 由 Google 积极开发，并且实际上是 Android 开发的官方 IDE。如果需要，我将把安装 IDE 和 SDK 留给读者作为练习。

要创建一个新项目，我们点击文件|新项目，并指定应用程序名称，公司域和项目位置，如下面的屏幕截图所示：

![](img/d5460c89-dad0-4a38-bd46-cf3412578f90.png)

接下来，我们需要指定要针对的 API 版本。这可能是一个棘手的选择。一方面，我们希望站在最前沿，并且拥有 Android 提供的所有出色的新功能，但另一方面，我们不希望针对一个如此新的 API 级别，以至于我们使应用程序对更多的 Android 用户而言无法使用（即无法卸载）。在这种情况下，Android 6.0，或者说 Marshmallow，似乎是一个可以接受的折衷方案：

![](img/0b9dd275-5991-41e4-be74-a93968b89653.png)

点击下一步，选择空白活动，下一步，完成，我们的项目已准备好开发。

在 Android 端，我们不会在用户界面方面做太多工作。一旦我们完成了项目，您可能会有各种各样的想法，这很好，但我们不会在这里花时间去做任何事情。也就是说，我们真正需要做的第一件事是请求用户权限以访问他们手机上的短信。

# 请求权限

在较早的 Android 版本中，权限是一种全有或全无的提议。但是，从 Android 6 开始，用户将被提示应用程序请求的每个权限，从而允许用户在拒绝其他权限的同时授予某些权限的可能性。我们需要请求一些权限--我们需要能够读取和写入短信，还需要访问联系人（这样我们就可以尝试弄清楚谁给我们发送了某条消息）。Android 提供了一个非常容易请求这些权限的 API，我们将把它放在我们的`onCreate()`方法中，如下所示：

```java
    public static final int PERMISSION_REQUEST_CODE = 42; 
    @Override 
    protected void onCreate(Bundle savedInstanceState) { 
      super.onCreate(savedInstanceState); 
     // ... 
     ActivityCompat.requestPermissions(this, 
            new String[]{ 
                    Manifest.permission.SEND_SMS, 
                    Manifest.permission.RECEIVE_SMS, 
                    Manifest.permission.READ_CONTACTS 
            }, 
            PERMISSION_REQUEST_CODE); 
    } 

```

当上述代码运行时，Android 将提示用户授予或拒绝请求的权限。这是异步完成的，因此在您的应用程序中，您需要确保在用户有机会授予权限之前，不要尝试执行任何需要您请求的权限的操作（并且，如果用户拒绝权限，则应用程序应该优雅地降级或失败）。

为了允许应用程序响应权限授予，Android 提供了一个回调。在我们的回调中，我们希望确保用户授予我们两个权限：

```java
    @Override 
    public void onRequestPermissionsResult(int requestCode, 
     String permissions[], int[] grantResults) { 
      switch (requestCode) { 
        case PERMISSION_REQUEST_CODE: { 
          if (grantResults.length != 3 
           || grantResults[0] !=  
                PackageManager.PERMISSION_GRANTED 
           || grantResults[1] !=  
                PackageManager.PERMISSION_GRANTED 
           || grantResults[2] !=  
                PackageManager.PERMISSION_GRANTED) { 
                  AlertDialog.Builder dialog =  
                    new AlertDialog.Builder(this); 
                  dialog.setCancelable(false); 
                  dialog.setTitle("Error"); 
                  dialog.setMessage("This app requires access
                   to text messages and contacts. Click OK
                   to close."); 
                  dialog.setPositiveButton("OK",  
                   new DialogInterface.OnClickListener() { 
                     @Override 
                     public void onClick(DialogInterface dialog,  
                      int id) { 
                        finish(); 
                      } 
                  }); 

                  final AlertDialog alert = dialog.create(); 
                  alert.show(); 
                } 
        } 
      } 
    } 

```

当 Android 回调到我们的应用程序时，我们需要确保`requestCode`是我们指定的--`PERMISSION_REQUEST_CODE`--以确保我们只响应我们自己的请求。

一旦我们确定了适当的响应，我们确保`grantResults`的长度是正确的，并且每个条目都是`PERMISSION_GRANTED`。如果数组太短，或者任一数组元素不是正确的类型，我们会显示一个对话框，通知用户需要两个权限，然后退出应用程序。

在我们的示例中，我们同时请求两个权限，因此我们同时响应两个权限。如果您有一组复杂的权限，例如，如果您的应用程序只能使用请求的某些权限，您可以多次调用`ActivityCompat.requestPermissions`，为每个请求提供一个不同的`requestCode`。然后，您需要扩展`onRequestPermissionsResult()`中的 switch 块，以涵盖每个新的`requestCode`。

最后关于权限的一句话。通常情况下，您应该始终检查是否具有执行给定任务所需的权限。您可以使用以下方法来做到这一点：

```java
    protected boolean checkPermission(Permissions permission) { 
      return ContextCompat.checkSelfPermission(this,  
        permission.permission) ==  
        PackageManager.PERMISSION_GRANTED; 
   } 

```

在我们的情况下，如果没有被授予所需的权限，我们就不允许应用程序运行，因此我们不需要担心额外的权限检查。

# 创建服务

项目的 Android 部分的核心是我们的 REST 端点。我们希望这些端点在手机开启时可用，因此我们不能使用`Activity`来托管它们。我们需要的是一个`Service`。Android 文档将`Service`定义为*可以在后台执行长时间操作的应用程序组件，并且不提供用户界面*。有三种类型的服务--`scheduled`（按计划运行），`started`（可以由另一个应用程序组件显式启动），和`bound`（通过`bindService()`调用绑定到应用程序组件，并在所有绑定的组件被销毁之前运行）。由于我们希望这个服务一直可用，我们需要一个已启动的服务。

要创建服务，点击文件 | 新建 | 服务 | 服务。输入`DeskDroidService`作为服务，取消选中 Exported，然后点击完成。这将为您提供以下存根代码：

```java
    public class DeskDroidService extends Service { 
      public DeskDroidService() { 
      } 

     @Override 
     public IBinder onBind(Intent intent) { 
       throw new UnsupportedOperationException( 
           "Not yet implemented"); 
     } 
    } 

```

向`AndroidManifest.xml`中添加以下内容：

```java
    <service 
      android:name=".DeskDroidService" 
      android:enabled="true" 
      android:exported="false" /> 

```

方法`onBind()`是抽象的，因此必须实现。我们不创建绑定服务，所以我们可以将其保持未实现，尽管我们将更改它以返回`null`而不是抛出异常。然而，我们对服务何时启动和停止感兴趣，因此我们需要重写这两个相关的生命周期方法：

```java
    public int onStartCommand(Intent intent, int flags, int startId) { 
      super.onStartCommand(intent, flags, startId); 
    }  
    public void onDestroy() { 
    } 

```

在这些方法中，我们将放置我们的 REST 服务代码。我们将再次使用 Jersey，JAX-RS 的参考实现，它提供了一种在 Java SE 环境中引导服务器的好方法，例如我们在 Android 应用程序中发现的环境。我们将把这个逻辑封装在一个名为`startServer()`的新方法中，如下所示：

```java
    protected static Server server; 
    protected void startServer() { 
      WifiManager WifiMgr = (WifiManager) getApplicationContext() 
       .getSystemService(Service.Wifi_SERVICE); 
      if (WifiMgr.isWifiEnabled()) { 
        String ipAddress = Formatter. 
         formatIpAddress(WifiMgr.getConnectionInfo() 
          .getIpAddress()); 
        URI baseUri = UriBuilder.fromUri("http://" + ipAddress) 
         .port(49152) 
         .build(); 
        ResourceConfig config =  
          new ResourceConfig(SseFeature.class) 
           .register(JacksonFeature.class); 
        server = JettyHttpContainerFactory.createServer(baseUri, 
         config); 
      } 
    } 

```

我们要做的第一件事是检查确保我们在 Wi-Fi 上。这并非绝对必要，但似乎是一个谨慎的预防措施，以防止应用程序监听连接，无论网络状态如何。如果手机不在 Wi-Fi 上，那么预期的笔记本电脑也很可能不在。然而，允许端点即使在蜂窝网络上也监听可能存在合法的用例。使这种限制可配置是一个很好的首选项驱动选项。

为了使这段代码工作，我们需要将这个新的权限添加到清单中：

```java
    <uses-permission android:name= 
      "android.permission.ACCESS_WIFI_STATE" /> 

```

一旦我们确定我们在 Wi-Fi 上，我们查找我们的 IP 地址，并引导一个基于 Jetty 的 Jersey 服务器。向值得尊敬的 Commodore 64 致敬，对于我们中年人来说，我们在 Wi-Fi 网络接口的端口`49152`上监听。

接下来，我们创建一个`ResourceConfig`实例，提供我们感兴趣的两个特性引用--`SseFeature`和`JacksonFeature`。我们已经见过`JacksonFeature`；这就是让我们能够使用 POJOs，将 JSON 问题留给 Jersey 的功能。但`SseFeature`是什么呢？

# 服务器发送事件

SSE，或者服务器发送事件，是一种我们可以从服务器向客户端流式传输数据的方式。通常，REST 请求的生命周期非常短暂--建立连接，发送请求，获取响应，关闭连接。然而，有时 REST 服务器可能在请求时没有客户端想要的所有数据（例如，从另一个数据源读取数据，比如日志文件或网络套接字）。因此，能够在数据可用时将数据推送到客户端将是很好的。这正是 SSE 允许我们做的。我们稍后会更详细地研究这个问题。

最后，我们通过调用`JettyHttpContainerFactory.createServer()`来启动服务器实例。由于我们需要能够稍后停止服务器，我们捕获服务器实例，并将其存储在一个实例变量中。我们从`onStartCommand()`中调用`startServer()`如下：

```java
    private static final Object lock = new Object(); 
    public int onStartCommand(Intent intent, int flags, int startId) { 
      super.onStartCommand(intent, flags, startId); 

      synchronized (lock) { 
        if (server == null) { 
          startServer(); 
          messageReceiver = new BroadcastReceiver() { 
            @Override 
            public void onReceive(Context context,  
             Intent intent) { 
               String code = intent.getStringExtra("code"); 
               DeskDroidService.this.code = code; 
               Log.d("receiver", "Got code: " + code); 
            } 
          }; 
          LocalBroadcastManager.getInstance(this). 
           registerReceiver( 
             messageReceiver,  
              new IntentFilter(CODE_GENERATED)); 
        } 
      } 

      return Service.START_STICKY; 
    } 

```

请注意，我们将对`startServer()`的调用包装在一个`synchronized`块中。对于可能不知道的人来说，`synchronized`是 Java 开发人员可以使用的更基本的并发代码方法之一。这个关键字的净效果是，试图执行这个代码块的多个线程必须同步执行，或者一个接一个地执行。我们这样做是为了确保如果有两个不同的进程尝试启动服务器，我们可以保证最多只有一个正在运行。如果没有这个块，第一个线程可能会启动服务器并将实例存储在变量中，而第二个线程可能会做同样的事情，但它的服务器实例存储在变量中，却无法启动。现在我们有一个正在运行的服务器，却没有有效的引用，所以我们无法停止它。

我们还注册了一个监听`CODE_GENERATED`的`BroadcastReceiver`。我们稍后会回来解释这一点，所以现在不用担心这个。

# 控制服务状态

如果我们现在运行应用程序，我们的服务不会运行，所以我们需要使它能够运行。我们将以几种不同的方式来做到这一点。第一种方式将是从我们的应用程序。我们希望确保在打开应用程序时服务正在运行，特别是在刚安装完应用程序后。为了做到这一点，我们需要在`MainActivity.onCreate()`中添加一行如下：

```java
    startService(new Intent(this, DeskDroidService.class)); 

```

现在应用程序启动时，它将保证服务正在运行。然而，我们不希望要求用户打开应用程序来运行服务。幸运的是，我们有一种方法可以在手机启动时启动应用程序。我们可以通过安装一个监听引导事件的`BroadcastReceiver`来实现，如下所示：

```java
    public class BootReceiver extends BroadcastReceiver { 
      @Override 
      public void onReceive(Context context, Intent intent) { 
        context.startService(new Intent(context,  
         DeskDroidService.class)); 
      } 
    } 

```

前面方法的主体与我们最近添加到`MainActivity`的内容相同。但是，我们需要注册服务，并请求权限。在`AndroidManifest.xml`中，我们需要添加这个：

```java
    <uses-permission android:name= 
      "android.permission.RECEIVE_BOOT_COMPLETED" /> 
    <receiver android:name=".BootReceiver" android:enabled="true"> 
      <intent-filter> 
        <action android:name= 
        "android.intent.action.BOOT_COMPLETED" /> 
      </intent-filter> 
    </receiver> 

```

现在我们有了一个服务，它可以在设备启动或应用程序启动时启动。然而，它并没有做任何有趣的事情，所以我们需要向服务器添加一些端点。

# 向服务器添加端点

如第九章中所述，*使用 Monumentum 做笔记*，JAX-RS 资源位于带有特定注解的 POJO 中。为了存根我们的端点类，我们可以从这里开始：

```java
    @Path("/") 
    @Produces(MediaType.APPLICATION_JSON) 
    protected class DeskDroidResource { 
    } 

```

我们还需要在 JAX-RS 中注册这个类，我们在`startServer()`中这样做：

```java
    config.registerInstances(new DeskDroidResource()); 

```

通常情况下，我们会将`DeskDroidResource.class`传递给`ResourceConfig`构造函数，就像我们使用`JacksonFeature.class`一样。我们将访问 Android 资源，为此，我们将需要`Service`的`Context`实例。互联网上有很多资源会建议创建一个自定义的`Application`类，并将其存储在`public static`中。虽然这似乎可以工作，但它也会泄漏内存，因此，例如，Android Studio 会在您尝试这样做时发出警告。然而，我们可以通过使用嵌套类来避免这种情况。这种方法可能有点笨拙，但我们的类应该足够小，以至于它仍然可以管理。

# 获取对话

让我们首先添加一个端点，以获取手机上的所有对话，如下所示：

```java
    @GET 
    @Path("conversations") 
    public Response getConversations() { 
      List<Conversation> conversations = new ArrayList<>(); 
      Cursor cur = getApplication().getContentResolver() 
      .query(Telephony.Sms.Conversations.CONTENT_URI,  
      null, null, null, null); 
      while (cur.moveToNext()) { 
        conversations.add(buildConversation(cur)); 
      } 

      Collections.sort(conversations, new ConversationComparator()); 

      return Response.ok(new GenericEntity<List<Conversation>>( 
      conversations) {}).build(); 
     } 

```

在这里，我们看到 Android 的构件开始出现--我们将使用`ContentProvider`来访问短信数据。`ContentProvider`是应用程序或在这种情况下是 Android 子系统以一种便携、存储无关的方式向外部消费者公开数据的一种方式。我们不关心数据是如何存储的。我们只是指定我们想要的字段、我们想要放在数据上的过滤器或限制条件，`ContentProvider`就会完成剩下的工作。

使用`ContentProvider`，我们不是通过表名来指定数据类型，就像我们在 SQL 中那样，而是通过`Uri`来指定。在这种情况下，我们指定`Telephony.Sms.Conversations.CONTENT_URI`。我们也将几个空值传递给`query()`。这些代表投影（或字段列表）、选择（或过滤器）、选择参数和排序顺序。由于这些都是`null`，我们希望提供程序中的每个字段和每一行都按自然排序顺序。这会得到一个`Cursor`对象，然后我们遍历它，创建`Conversation`对象，并将它们添加到我们的`List`中。

我们使用这种方法创建`Conversation`实例：

```java
    private Conversation buildConversation(Cursor cur) { 
      Conversation conv = new Conversation(); 
      final int threadId =  
        cur.getInt(cur.getColumnIndex("thread_id")); 
      conv.setThreadId(threadId); 
      conv.setMessageCount( 
        cur.getInt(cur.getColumnIndex("msg_count"))); 
      conv.setSnippet(cur.getString(cur.getColumnIndex("snippet"))); 
      final List<Message> messages =  
        getSmsMessages(conv.getThreadId()); 
      Set<String> participants = new HashSet<>(); 
      for (Message message : messages) { 
        if (!message.isMine()) { 
          participants.add(message.getAddress()); 
        } 
      } 
      conv.setParticipants(participants); 
      conv.setMessages(messages); 
      return conv; 
    } 

```

每个对话只是一个线程 ID、消息计数和片段，这是最后收到的消息。要获取实际的消息，我们调用`getSmsMessages()`如下：

```java
    private List<Message> getSmsMessages(int threadId) { 
      List<Message> messages = new ArrayList<>(); 
      Cursor cur = null; 
      try { 
        cur = getApplicationContext().getContentResolver() 
         .query(Telephony.Sms.CONTENT_URI, 
         null, "thread_id = ?", new String[] 
         {Integer.toString(threadId)}, 
         "date DESC"); 

        while (cur.moveToNext()) { 
          Message message = new Message(); 
          message.setId(cur.getInt(cur.getColumnIndex("_id"))); 
          message.setThreadId(cur.getInt( 
            cur.getColumnIndex("thread_id"))); 
          message.setAddress(cur.getString( 
            cur.getColumnIndex("address"))); 
          message.setBody(cur.getString( 
            cur.getColumnIndexOrThrow("body"))); 
          message.setDate(new Date(cur.getLong( 
            cur.getColumnIndexOrThrow("date")))); 
          message.setMine(cur.getInt( 
            cur.getColumnIndex("type")) ==  
              Telephony.Sms.MESSAGE_TYPE_SENT); 
          messages.add(message); 
        } 
      } catch (Exception e) { 
          e.printStackTrace(); 
      } finally { 
          if (cur != null) { 
            cur.close(); 
          } 
      } 
      return messages; 
    } 

```

这种方法和处理逻辑与对话的大部分相同。当然，`ContentProvider`的`Uri`，`Telephony.Sms.CONTENT_URI`是不同的，我们为查询指定了一个过滤器，如下所示：

```java
    cur = getApplicationContext().getContentResolver().query( 
      Telephony.Sms.CONTENT_URI, 
       null, "thread_id = ?", new String[] 
       {Integer.toString(threadId)}, 
       "date DESC"); 

```

我们在这里有一点数据分析。我们需要知道哪些消息是我们发送的，哪些是我们接收的，以便我们可以更有意义地显示对话。在设备上，我们发送的消息的类型是`Telephony.Sms.MESSAGE_TYPE_SENT`。该字段的值大致对应于文件夹（已发送、已接收、草稿等）。我们没有通过共享常量的值将 Android API 的一部分泄漏到我们的 API 中，而是有一个`boolean`字段`isMine`，如果消息的类型是`MESSAGE_TYPE_SENT`，则为 true。这是一个稍微笨拙的替代方法，但它有效并且应该足够清晰。

一旦我们返回消息列表，我们遍历列表，获取唯一参与者的列表（应该只有一个，因为我们处理的是短信消息）。

最后，我们使用 Jersey 的 POJO 映射功能将`List<Conversation>`返回给客户端，如下所示：

```java
    return Response.ok(new GenericEntity<List<Conversation>>( 
      conversations) {}).build();

```

![](img/c65074a9-60d0-4ce9-9fbd-cac745d5fae5.png)

如果我们点击运行或调试按钮（工具栏中的大三角形或三角形上的带有错误图标），您将被要求选择部署目标，如下截图所示：

![](img/666baa0f-7732-4d02-9af4-3c0c135d3e34.png)

由于我们需要 Wi-Fi，我选择了我的物理设备。如果您想配置一个带有 Wi-Fi 的模拟器，那也可以。点击确定，几秒钟后，应用程序应该在您选择的设备上启动，然后我们可以按照以下方式进行第一个 REST 请求：

```java
    $ curl http://192.168.0.2:49152/conversations | jq . 
    [ 
    { 
      "messageCount": 2, 
      "messages": [ 
        { 
          "address": "5551234567", 
          "body": "Demo message", 
          "date": 1493269498618, 
          "id": 301, 
          "mine": true, 
          "threadId": 89 
        }, 
        { 
          "address": "+15551234567", 
          "body": "Demo message", 
          "date": 1493269498727, 
          "id": 302, 
          "mine": false, 
          "threadId": 89 
        } 
      ], 
      "participants": [ "+15551234567" ], 
      "snippet": "Demo message", 
      "threadId": 89 
    } 
    ] 

```

这段代码显示了我和自己的对话。也许是太多的深夜，但你可以看到第一条消息，最老的消息，标记为我的，这是我发给自己的，第二条是我收到回复的地方。非常酷，但是如何发送消息呢？事实证明，这实际上非常简单。

# 发送短信

为了发送消息，我们将创建一个 POST 端点，该端点接受一个`Message`对象，然后我们将其拆分并传递给安卓的电话 API。

```java
    @POST 
    @Path("conversations") 
    public Response sendMessage(Message message)  
    throws InterruptedException { 
       final SmsManager sms = SmsManager.getDefault(); 
       final ArrayList<String> parts =  
       sms.divideMessage(message.getBody()); 
       final CountDownLatch sentLatch =  
       new CountDownLatch(parts.size()); 
       final AtomicInteger statusCode = new AtomicInteger( 
       Response.Status.CREATED.getStatusCode()); 
       final BroadcastReceiver receiver = new BroadcastReceiver() { 
       @Override 
       public void onReceive(Context context, Intent intent) { 
            if (getResultCode() != Activity.RESULT_OK) { 
                    statusCode.set( 
                        Response.Status.INTERNAL_SERVER_ERROR 
                            .getStatusCode()); 
            } 
             sentLatch.countDown(); 
          } 
        }; 
      registerReceiver(receiver,  
      new IntentFilter("com.steeplesoft.deskdroid.SMS_SENT")); 
      ArrayList<PendingIntent> sentPIs = new ArrayList<>(); 
      for (int i = 0; i < parts.size(); i++) { 
         sentPIs.add(PendingIntent.getBroadcast( 
            getApplicationContext(), 0, 
            new Intent("com.steeplesoft.deskdroid.SMS_SENT"), 0)); 
      } 
      sms.sendMultipartTextMessage(message.getAddress(), null,  
      parts, sentPIs, null); 

      sentLatch.await(5, TimeUnit.SECONDS); 
      unregisterReceiver(receiver); 
      return Response.status(statusCode.get()).build(); 
     } 

```

这种方法有很多内容。以下是详细说明：

1.  我们获得了`SmsManager`类的引用。这个类将为我们做所有的工作。

1.  我们要求`SmsManager`为我们分割消息。文本消息通常限制在 160 个字符，所以这将根据需要分割消息。

1.  我们创建了一个与消息中部分数量相匹配的`CountDownLatch`。

1.  我们创建了一个`AtomicInteger`来存储状态码。正如我们将在一会儿看到的，我们需要从匿名类内部改变这个变量的值。然而，匿名类要访问其封闭范围的变量，这些变量必须是`final`的，这意味着我们不能有一个`final int`，因为那样我们就无法改变值。不过，使用`AtomicInteger`，我们可以调用`set()`来改变值，同时保持实例引用不变，这就是变量将持有的内容。

1.  我们创建了一个新的`BroadcastReceiver`，它将处理消息发送时的`Intent`广播（我们将在后面看到）。在`onReceive()`中，如果结果代码不是`ACTIVITY.RESULT_OK`，我们调用`AtomicInteger.set()`来反映失败。然后我们调用`sentLatch.countDown()`来表示这个消息部分已经被处理。

1.  通过调用`registerReceiver()`，我们让操作系统知道我们的新接收器。我们提供一个`IntentFilter`来限制我们的接收器必须处理哪些`Intent`。

1.  然后我们为消息的每个部分创建一个新的`PendingIntent`。这将允许我们对每个部分的发送尝试做出反应。

1.  我们调用`sendMultipartTextMessage()`来发送消息的部分。安卓会为我们处理多部分消息的细节，所以不需要额外的努力。

1.  我们需要等待所有消息部分被发送，所以我们调用`sentLatch.await()`来给系统发送消息的时间。然而，我们不想永远等下去，所以我们给了它一个五秒的超时时间，这应该足够长了。可以想象，有些网络可能在发送短信方面非常慢，所以这个值可能需要调整。

1.  一旦我们通过了门闩，我们就`unregister`我们的接收器，并返回状态码。

再次使用 curl，我们现在可以测试发送消息（确保再次点击`Run`或`Debug`来部署您更新的代码）：

```java
        $ curl -v -X POST -H 'Content-type: application/json'
        http://192.168.0.2:49152/conversations -d 
        '{"address":"++15551234567", "body":"Lorem ipsum dolor sit 
         amet..."}' 
        > POST /conversations HTTP/1.1 
        > Content-type: application/json 
        > Content-Length: 482 
        < HTTP/1.1 201 Created 

```

在前面的`curl`中，我们向接收者发送了一些`lorem ipsum`文本，这给了我们一个漂亮的、长长的消息（请求负载总共 482 个字符），这些消息被正确地分块并发送到目标电话号码，正如`201 Created`响应状态所示。

现在我们在手机上有一个工作的 REST 服务，它让我们读取现有的消息并发送新消息。使用`curl`与服务交互已经足够好了，但是现在是时候建立我们的桌面客户端，并为这个项目打造一个漂亮的外观了。

# 创建桌面应用程序

为了构建我们的应用程序，我们将返回 NetBeans 和 JavaFX。和之前的章节一样，我们将通过点击文件|新建项目来创建一个基于 Maven 的 JavaFX 应用程序：

![](img/665acc29-dbe6-41a0-ac15-afcd98ae6b23.png)

在下一步中，将项目命名为`deskdroid-desktop`，验证包名称，并单击完成。虽然不是严格必要的，但让我们稍微清理一下命名，将控制器更改为`DeskDroidController`，将 FXML 文件更改为`deskdroid.fxml`。我们还需要修改控制器中对 FXML 和 CSS 的引用，以及在 FXML 中对控制器的引用。单击运行|运行项目，以确保一切连接正确。一旦应用程序启动，我们可以立即关闭它，以便开始进行更改。

# 定义用户界面

让我们开始构建用户界面。应用程序将如下所示：

![](img/c41ac1fa-9371-4d5a-bfe7-26fd8fb937a9.png)

在前面的屏幕中，我们将在左侧显示我们的对话列表，并在右侧显示所选对话。我们将添加一个自动刷新的机制，但如果需要，刷新对话将允许手动刷新。新消息应该是不言而喻的。

当然，我们可以使用 Gluon 的 Scene Builder 来构建用户界面，但让我们来看看 FXML。我们将像往常一样从`BorderPane`开始，如下所示：

```java
    <BorderPane fx:id="borderPane" minWidth="1024" prefHeight="768"  

    fx:controller="com.steeplesoft.deskdroid.
    desktop.DeskDroidController"> 

```

对于`top`部分，我们将添加一个菜单栏，如下所示：

```java
    <MenuBar BorderPane.alignment="CENTER"> 
      <menus> 
        <Menu text="_File"> 
            <items> 
                <MenuItem onAction="#connectToPhone"  
                    text="_Connect to Phone" /> 
                <MenuItem onAction="#disconnectFromPhone"  
                    text="_Disconnect from Phone" /> 
                <MenuItem onAction="#closeApplication"  
                    text="E_xit"> 
                    <accelerator> 
                        <KeyCodeCombination alt="ANY" code="F4"  
                            control="UP" meta="UP" shift="UP"  
                            shortcut="UP" /> 
                    </accelerator> 
                </MenuItem> 
              </items> 
          </Menu> 
       </menus> 
    </MenuBar> 

```

在`FileMenu`中，我们将有三个`MenuItem`：`connectToPhone`，`disconnectFromPhone`和`Exit`。每个菜单项都将有一个助记符，如下划线所示。`ExitMenuItem`有一个加速键`ALT-F4`。

我们将把大部分用户界面放在`center`部分。垂直分割允许我们调整用户界面的两侧。为此，我们使用`SplitPane`如下所示：

```java
    <center> 
      <SplitPane dividerPositions="0.25"  
        BorderPane.alignment="CENTER"> 
      <items> 

```

使用`dividerPositions`，我们将默认分割设置为水平规则的 25%标记。`SplitPane`有一个嵌套的`items`元素来保存其子元素，我们将左侧元素`ListView`添加到其中：

```java
    <VBox> 
      <children> 
        <ListView fx:id="convList" VBox.vgrow="ALWAYS" /> 
      </children> 
    </VBox> 

```

我们将`ListView`包装在`VBox`中，以便更容易地使`ListView`根据需要增长和收缩。

最后，让我们构建用户界面的右侧：

```java
     <VBox fx:id="convContainer"> 
       <children> 
        <HBox> 
            <children> 
                <Button mnemonicParsing="false"  
                        onAction="#refreshConversations"  
                        text="Refresh Conversations"> 
                    <HBox.margin> 
                        <Insets right="5.0" /> 
                    </HBox.margin> 
                </Button> 
                <Button fx:id="newMessageBtn"  
                    text="New Message" /> 
            </children> 
            <padding> 
                <Insets bottom="5.0" left="5.0"  
                    right="5.0" top="5.0" /> 
            </padding> 
        </HBox> 
        <ListView fx:id="messageList" VBox.vgrow="ALWAYS" /> 
      </children> 
    </VBox> 

```

在右侧，我们还有一个`VBox`，用来安排我们的两个用户界面元素。第一个是`HBox`，其中包含两个按钮：刷新对话和新消息。第二个是我们的`ListView`，用于显示所选对话。

# 定义用户界面行为

虽然我们可以在 FXML 中定义用户界面的结构，但除了最简单的应用程序外，用户界面仍然需要一些 Java 代码来完成其行为的定义。我们将在`DeskDroidController.initialize()`中进行。我们将从用户界面的左侧，对话列表开始，如下所示：

```java
    @FXML 
    private ListView<Conversation> convList; 
    private final ObservableList<Conversation> conversations =  
    FXCollections.observableArrayList(); 
    private final SimpleObjectProperty<Conversation> conversation =  
    new SimpleObjectProperty<>(); 
    @Override 
    public void initialize(URL url, ResourceBundle rb) { 
      convList.setCellFactory(list ->  
      new ConversationCell(convList)); 
      convList.setItems(conversations); 
       convList.getSelectionModel().selectedItemProperty() 
            .addListener((observable, oldValue, newValue) -> { 
                conversation.set(newValue); 
                messages.setAll(newValue.getMessages()); 
                messageList.scrollTo(messages.size() - 1); 
     }); 

```

我们声明一个可注入的变量来保存对我们的`ListView`的引用。JavaFX 将为我们设置该值，感谢注解`@FXML`。`ListView`将需要一个要显示的模型，我们将其声明为`conversations`，并声明`conversation`来保存当前选定的对话。

在`initialize()`方法中，我们将所有内容连接在一起。由于`ListView`将显示我们的领域对象，我们需要为其声明一个`CellFactory`，我们使用传递给`setCellFactory()`的 lambda 来实现。我们稍后会看一下`ListCell`。

接下来，我们将`ListView`与其模型`conversations`关联，并定义实际上是一个`onClick`监听器。我们通过向`ListView`的`SelectionModel`添加监听器来实现这一点。在该监听器中，我们更新当前选定的对话，更新消息`ListView`以显示对话，并将该`ListView`滚动到最底部，以便看到最近的消息。

初始化消息`ListView`要简单得多。我们需要这些实例变量：

```java
    @FXML 
    private ListView<Message> messageList; 
    private final ObservableList<Message> messages =  
    FXCollections.observableArrayList(); 

```

我们还需要在`initialize()`中添加这些行：

```java
    messageList.setCellFactory(list -> new MessageCell(messageList)); 
    messageList.setItems(messages); 

```

新消息按钮需要一个处理程序：

```java
    newMessageBtn.setOnAction(event -> sendNewMessage()); 

```

`ConversationCell`告诉 JavaFX 如何显示`Conversation`实例。为此，我们创建一个新的`ListCell`子元素，如下所示：

```java
    public class ConversationCell extends ListCell<Conversation> { 

```

然后我们重写`updateItem()`：

```java
    @Override 
    protected void updateItem(Conversation conversation,  
    boolean empty) { 
    super.updateItem(conversation, empty); 
    if (conversation != null) { 
        setWrapText(true); 
        final Participant participant =  
            ConversationService.getInstance() 
                .getParticipant(conversation 
                    .getParticipant()); 
        HBox hbox = createWrapper(participant); 

        hbox.getChildren().add( 
            createConversationSnippet(participant,  
                conversation.getSnippet())); 
        setGraphic(hbox); 
     } else { 
        setGraphic(null); 
     } 
    } 

```

如果单元格给定了“对话”，我们会处理它。如果没有，我们将单元格的图形设置为 null。如果我们无法做到这一点，当浏览列表时，将会产生不可预测的结果。

要构建单元格内容，我们首先获取`Participant`，然后创建包装组件，如下所示：

```java
    protected HBox createWrapper(final Participant participant) { 
      HBox hbox = new HBox(); 
      hbox.setManaged(true); 
      ImageView thumbNail = new ImageView(); 
      thumbNail.prefWidth(65); 
      thumbNail.setPreserveRatio(true); 
      thumbNail.setFitHeight(65); 
      thumbNail.setImage(new Image( 
        ConversationService.getInstance() 
           .getParticipantThumbnail( 
               participant.getPhoneNumber()))); 
      hbox.getChildren().add(thumbNail); 
      return hbox; 
    } 

```

这是相当标准的 JavaFX 内容——创建一个`HBox`，并向其中添加一个`ImageView`。不过，我们使用了一个我们尚未看过的类——`ConversationService`。稍后我们会看到这个，但现在知道我们将在这个类中封装我们的 REST 调用就足够了。在这里，我们调用一个端点（我们尚未看到的）来获取对话另一端的电话号码的联系信息。

我们还需要创建对话片段，如下所示：

```java
    protected VBox createConversationSnippet( 
     final Participant participant, String snippet) { 
      VBox vbox = new VBox(); 
      vbox.setPadding(new Insets(0, 0, 0, 5)); 
      Label sender = new Label(participant.getName()); 
      sender.setWrapText(true); 
      Label phoneNumber = new Label(participant.getPhoneNumber()); 
      phoneNumber.setWrapText(true); 
      Label label = new Label(snippet); 
      label.setWrapText(true); 
      vbox.getChildren().addAll(sender, phoneNumber, label); 
      return vbox; 
    } 

```

使用`VBox`来确保垂直对齐，我们创建两个标签，一个包含参与者的信息，另一个包含对话片段。

虽然这完成了单元格定义，但如果我们现在运行应用程序，`ListCell`的内容可能会被`ListView`本身的边缘裁剪。例如，查看以下截图中顶部列表和底部列表之间的差异：

![](img/4907b58e-45dd-4ce7-a9e2-b2bb7a4ecef7.png)

为了使我们的`ListCell`的行为与上一个屏幕底部看到的一样，我们需要对我们的代码进行一些更改，如下所示：

```java
    public ConversationCell(ListView list) { 
      super(); 
      prefWidthProperty().bind(list.widthProperty().subtract(2)); 
      setMaxWidth(Control.USE_PREF_SIZE); 
    } 

```

在我们之前的`CellFactory`中，我们传入了对封闭`ListView`的引用。

```java
    convList.setCellFactory(list -> new ConversationCell(convList)); 

```

然后在构造函数中，我们将单元格的首选宽度绑定到列表的实际宽度（并减去一小部分以调整控件边框）。现在渲染时，我们的单元格将如我们所期望的那样换行。

`MessageCell`的定义类似，如下所示：

```java
    public class MessageCell extends ListCell<Message> { 
      public MessageCell(ListView list) { 
          prefWidthProperty() 
            .bind(list.widthProperty().subtract(20)); 
          setMaxWidth(Control.USE_PREF_SIZE); 
      } 

    @Override 
    public void updateItem(Message message, boolean empty) { 
        super.updateItem(message, empty); 
        if (message != null && !empty) { 
            if (message.isMine()) { 
                wrapMyMessage(message); 
            } else { 
                wrapTheirMessage(message); 
            } 
         } else { 
            setGraphic(null); 
        } 
    } 

```

对于*我的*消息，我们以这种方式创建内容：

```java
    private static final SimpleDateFormat DATE_FORMAT =  
     new SimpleDateFormat("EEE, MM/dd/yyyy hh:mm aa"); 
    private void wrapMyMessage(Message message) { 
     HBox hbox = new HBox(); 
     hbox.setAlignment(Pos.TOP_RIGHT); 
     createMessageBox(message, hbox, Pos.TOP_RIGHT); 
     setGraphic(hbox); 
    } 
    private void createMessageBox(Message message, Pane parent,  
     Pos alignment) { 
       VBox vbox = new VBox(); 
       vbox.setAlignment(alignment); 
       vbox.setPadding(new Insets(0,0,0,5)); 
       Label body = new Label(); 
       body.setWrapText(true); 
       body.setText(message.getBody()); 

       Label date = new Label(); 
       date.setText(DATE_FORMAT.format(message.getDate())); 

       vbox.getChildren().addAll(body,date); 
       parent.getChildren().add(vbox); 
    } 

```

**消息框**与之前的对话片段非常相似——消息的垂直显示，后跟其日期和时间。这种格式将被用于*我的*消息和*他们*的消息，因此我们使用`javafx.geometry.Pos`将控件对齐到右侧或左侧。

*他们*的消息是这样创建的：

```java
    private void wrapTheirMessage(Message message) { 
      HBox hbox = new HBox(); 
      ImageView thumbNail = new ImageView(); 
      thumbNail.prefWidth(65); 
      thumbNail.setPreserveRatio(true); 
      thumbNail.setFitHeight(65); 
      thumbNail.setImage(new Image( 
            ConversationService.getInstance() 
                .getParticipantThumbnail( 
                    message.getAddress()))); 
      hbox.getChildren().add(thumbNail); 
      createMessageBox(message, hbox, Pos.TOP_LEFT); 
      setGraphic(hbox); 
   } 

```

这与*我的*消息类似，唯一的区别是，如果与手机上的联系人关联的话，我们会显示发送者的个人资料图片，这些图片是通过`ConversationService`类从手机中检索到的。

我们还有更多的工作要做，但这就是应用程序在有数据时的样子：

![](img/9bdc898a-39b4-4c3f-9a2c-f2b87babafa9.png)

要获取数据，我们需要一个 REST 客户端，这在`ConversationService`中找到：

```java
    public class ConversationService { 
      public static class LazyHolder { 
        public static final ConversationService INSTANCE =  
            new ConversationService(); 
      } 

     public static ConversationService getInstance() { 
        return LazyHolder.INSTANCE; 
      } 
     private ConversationService() { 
        Configuration configuration = new ResourceConfig() 
                .register(JacksonFeature.class) 
                .register(SseFeature.class); 
        client = ClientBuilder.newClient(configuration); 
     } 

```

使用所谓的“按需初始化持有者”习语，我们创建了一种贫民的单例。由于构造函数是私有的，因此无法从此类外部调用。嵌套的静态类`LazyHolder`只有在最终被引用时才会被初始化，这发生在第一次调用`getInstance()`时。一旦调用了该方法，`LazyHolder`就会被加载和初始化，此时构造函数会运行。创建的实例存储在静态变量中，并在 JVM 运行时存在。每次后续调用都将返回相同的实例。这对我们来说很重要，因为我们有一些昂贵的对象需要创建，以及类中的一些简单缓存：

```java
    protected final Client client; 
    protected final Map<String, Participant> participants =  
      new HashMap<>(); 

```

在上述代码中，我们初始化了客户端实例，注册了`JacksonFeature`，这使我们得到了已经讨论过的 POJO 映射。我们还注册了`SseFeature`，这是 Jersey 的一个更高级的功能，我们稍后会详细讨论。

我们已经看到了对话列表。这是使用此方法的数据生成的：

```java
    public List<Conversation> getConversations() { 
      List<Conversation> list; 
      try { 
       list = getWebTarget().path("conversations") 
                .request(MediaType.APPLICATION_JSON) 
                .header(HttpHeaders.AUTHORIZATION,  
                    getAuthorizationHeader()) 
                .get(new GenericType<List<Conversation>>() {}); 
       } catch (Exception ce) { 
        list = new ArrayList<>(); 
      } 
      return list; 
    } 
    public WebTarget getWebTarget() { 
    return client.target("http://" 
            + preferences.getPhoneAddress() + ":49152/"); 
    } 

```

`WebTarget`是一个 JAX-RS 类，表示由资源 URI 标识的*资源目标*。我们从偏好设置中获取电话地址，稍后我们会讨论这个问题。一旦我们有了我们的`WebTarget`，我们通过附加`conversations`来完成构建 URI，指定请求的 mime 类型，并发出`GET`请求。请注意，我们的请求在这里有点乐观，因为我们没有进行任何状态码检查。如果抛出`Exception`，我们只是返回一个空的`List`。

我们看到的另一个方法是`getParticipant()`，如下所示：

```java
    public Participant getParticipant(String number) { 
      Participant p = participants.get(number); 
      if (p == null) { 
        Response response = getWebTarget() 
                .path("participants") 
                .path(number) 
                .request(MediaType.APPLICATION_JSON) 
                .header(HttpHeaders.AUTHORIZATION,  
                    getAuthorizationHeader()) 
                .get(Response.class); 
        if (response.getStatus() == 200) { 
            p = response.readEntity(Participant.class); 
            participants.put(number, p); 
            if (p.getThumbnail() != null) { 
                File thumb = new File(number + ".png"); 
                try (OutputStream stream =  
                        new FileOutputStream(thumb)) { 
                    byte[] data = DatatypeConverter 
                        .parseBase64Binary(p.getThumbnail()); 
                    stream.write(data); 
                } catch (IOException e) { 
                    e.printStackTrace(); 
                } 
             } 
          } 
       } 
     return p; 
   } 

```

在最后一个方法中，我们看到了我们的缓存发挥作用。当请求`Participant`时，我们查看是否已经获取了这些信息。如果是，我们返回缓存的信息。如果没有，我们可以请求它。

与`getConversations()`类似，我们为适当的端点构建请求，并发送`GET`请求。不过这次，我们确实检查状态码。只有状态码为`200（OK）`时，我们才继续处理响应。在这种情况下，我们要求 JAX-RS 返回`Participant`实例，`JacksonFeature`愉快地为我们从 JSON 响应体中构建，并立即添加到我们的缓存中。

如果服务器找到联系人的缩略图，我们需要处理它。服务器部分，我们将在讨论完这个方法后立即查看，将缩略图作为 base 64 编码的字符串发送到 JSON 对象的主体中，因此我们将其转换回二进制表示，并将其保存到文件中。请注意，我们使用了 try-with-resources，因此我们不需要担心清理工作。

```java
    try (OutputStream stream = new FileOutputStream(thumb)) 

```

我们还没有看到此操作的服务器端，所以现在让我们来看看。在 Android Studio 中的我们的 Android 应用中，我们在`DeskDroidResource`上有这个方法：

```java
    @GET 
    @Path("participants/{address}") 
    public Response getParticipant(@PathParam("address")  
    String address) { 
      Participant p = null; 
      try { 
        p = getContactsDetails(address); 
        } catch (IOException e) { 
        return Response.serverError().build(); 
       } 
      if (p == null) { 
        return Response.status(Response.Status.NOT_FOUND).build(); 
       } else { 
        return Response.ok(p).build(); 
       } 
    } 

```

我们尝试构建`Participant`实例。如果抛出异常，我们返回`500`（服务器错误）。如果返回`null`，我们返回`404`（未找到）。如果找到参与者，我们返回`200`（OK）和参与者。

要构建参与者，我们需要查询电话联系人。这与短信查询的方式非常相似：

```java
    protected Participant getContactsDetails(String address) throws 
     IOException { 
      Uri contactUri = Uri.withAppendedPath( 
        ContactsContract.PhoneLookup.CONTENT_FILTER_URI,  
        Uri.encode(address)); 
        Cursor phones = deskDroidService.getApplicationContext() 
        .getContentResolver().query(contactUri, 
        new String[]{ 
          ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME, 
          "number", 
          ContactsContract.CommonDataKinds.Phone 
            .PHOTO_THUMBNAIL_URI}, 
            null, null, null); 
        Participant participant = new Participant(); 
        if (phones.moveToNext()) { 
          participant.setName(phones.getString(phones 
          .getColumnIndex( 
          ContactsContract.CommonDataKinds.Phone 
           .DISPLAY_NAME))); 
          participant.setPhoneNumber(phones.getString( 
            phones.getColumnIndex("number"))); 
          String image_uri = phones.getString( 
            phones.getColumnIndex( 
              ContactsContract.CommonDataKinds.Phone 
               .PHOTO_THUMBNAIL_URI)); 
          if (image_uri != null) { 
            try (InputStream input = deskDroidService 
              .getApplicationContext().getContentResolver() 
              .openInputStream(Uri.parse(image_uri)); 
            ByteArrayOutputStream buffer = 
              new ByteArrayOutputStream()) { 
                int nRead; 
                byte[] data = new byte[16384]; 

                while ((nRead = input.read(data, 0,  
                        data.length)) != -1) { 
                    buffer.write(data, 0, nRead); 
                } 

                buffer.flush(); 
                participant.setThumbnail(Base64 
                    .encodeToString(buffer.toByteArray(),  
                        Base64.DEFAULT)); 
            } catch (IOException e) { 
                e.printStackTrace(); 
              } 
            } 
        } 
        phones.close(); 
        return participant; 
    } 

```

前面的内容与我们之前在对话和游标管理中看到的相同，但有一个例外。如果联系人有缩略图，查询将返回该图像的`Uri`。我们可以使用`ContentResolver`打开一个`InputStream`，使用该`Uri`读取内容，然后将其加载到`ByteArrayOutputStream`中。使用 Android 的`Base64`类，我们将这个二进制图像编码为一个`String`，并将其添加到我们的`Participant`模型中。我们之前看到了这个操作的解码部分。

# 发送消息

现在我们可以看到我们一直在进行的对话，我们需要添加参与这些对话的能力--发送新的文本消息。我们将从客户端开始。我们实际上已经看到了分配给`New Message`按钮的处理程序。它如下所示：

```java
    newMessageBtn.setOnAction(event -> sendNewMessage()); 

```

现在我们需要看看这个`sendNewMessage()`方法本身：

```java
    private void sendNewMessage() { 
      Optional<String> result = SendMessageDialogController 
        .showAndWait(conversation.get()); 
      if (result.isPresent()) { 
        Conversation conv = conversation.get(); 
        Message message = new Message(); 
        message.setThreadId(conv.getThreadId()); 
        message.setAddress(conv.getParticipant()); 
        message.setBody(result.get()); 
        message.setMine(true); 
        if (cs.sendMessage(message)) { 
            conv.getMessages().add(message); 
            messages.add(message); 
        } else { 
            Alert alert = new Alert(AlertType.ERROR); 
            alert.setTitle("Error"); 
            alert.setHeaderText( 
                "An error occured while sending the message."); 
            alert.showAndWait(); 
        } 
      } 
    } 

```

实际对话显示在另一个窗口中，所以我们有一个单独的 FXML 文件`message_dialog.fxml`和控制器`SendMessageDialogController`。对话框关闭时，我们检查返回的`Optional`，看用户是否输入了消息。如果是，按以下方式处理消息：

1.  获取所选`Conversation`的引用。

1.  创建一条新消息，设置会话 ID，收件人和正文。

1.  使用`ConversationService`，我们尝试发送消息：

1.  如果成功，我们用新消息更新用户界面。

1.  如果不成功，我们显示一个错误消息。

`SendMessageController`的工作方式与我们之前看过的其他控制器一样。最有趣的是`showAndWait()`方法。我们将使用该方法显示对话框，等待其关闭，并将任何用户响应返回给调用者。对话框如下所示：

![](img/00f5c6b8-1902-4dba-819e-22097a7c7659.png)

该方法如下所示：

```java
    public static Optional<String> showAndWait( 
      Conversation conversation) { 
      try { 
        FXMLLoader loader =  
            new FXMLLoader(SendMessageDialogController.class 
                .getResource("/fxml/message_dialog.fxml")); 
        Stage stage = new Stage(); 
        stage.setScene(new Scene(loader.load())); 
        stage.setTitle("Send Text Message"); 
        stage.initModality(Modality.APPLICATION_MODAL); 
        final SendMessageDialogController controller =  
            (SendMessageDialogController) loader.getController(); 
        controller.setConversation(conversation); 
        stage.showAndWait(); 
        return controller.getMessage(); 
      } catch (IOException ex) { 
          throw new RuntimeException(ex); 
      } 
    } 

```

前面方法中的前几行是我们通常看到的，即创建加载器和`Stage`。在显示`Stage`之前，我们设置模态，并传入当前的`Conversation`。最后，我们调用`showAndWait()`，在这一点上，该方法会阻塞，直到用户关闭对话框，然后我们返回输入的消息：

```java
    public Optional<String> getMessage() { 
      return Optional.ofNullable(message); 
    } 

```

Java 的`Optional`是一个*可能包含非空值的容器对象*。`message`的值可能设置或未设置，这取决于对话框中点击了哪个按钮。使用`Optional`，我们可以返回一个可能为空的值，并在调用者中更安全地处理它--`if (result.isPresent())`。

消息的发送是`ConversationService`中的一个简单的 POST 操作，如下所示：

```java
    public boolean sendMessage(Message message) { 
      Response r = getWebTarget().path("conversations") 
        .request() 
        .header(HttpHeaders.AUTHORIZATION, 
          getAuthorizationHeader()) 
        .post(Entity.json(message)); 
      return r.getStatus() == Response.Status.CREATED 
        .getStatusCode(); 
    } 

```

客户端很简单，但服务器端呢？毫不奇怪，复杂性就在那里：

```java
    @POST 
    @Path("conversations") 
    public Response sendMessage(Message message) throws
    InterruptedException { 
      final SmsManager sms = SmsManager.getDefault(); 
      final ArrayList<String> parts =  
        sms.divideMessage(message.getBody()); 

```

要添加端点，我们使用正确的注释定义一个新的方法。这个方法将在路径`conversations`上监听`POST`请求，并期望`Message`作为其有效负载。发送消息的实际工作由`SmsManager`处理，因此我们获取对默认管理器的引用。下一步调用`divideMessage()`，但这是怎么回事呢？

短信技术上限制为 160 个字符。Twitter 用户可能已经对此有所了解。Twitter 将推文限制为 140 个字符，留下 20 个字符供发送者的名称。虽然 Twitter 严格遵守这一限制，但普通短信用户有更好的体验。如果消息超过 160 个字符，大多数现代手机在发送时会将消息分成 153 个字符的段（使用 7 个字符的分段信息将这些段拼接在一起），如果手机支持的话，在接收端将这些段合并成一条消息。`SmsManager` API 通过`divideMessage()`为我们处理了这种复杂性。

一旦消息被*分块*，我们的工作就变得有点困难。我们希望能够返回一个状态码，指示消息是否成功发送。为了做到这一点，我们需要检查消息的每个块的状态，无论是一个还是十个。使用`SmsManager`发送文本消息时，Android 会广播带有结果的`Intent`。为了对此做出反应，我们需要注册一个接收器。将所有这些放在一起，我们得到了这段代码：

```java
    final CountDownLatch sentLatch = new CountDownLatch(parts.size()); 
    final AtomicInteger statusCode = 
      new AtomicInteger( 
        Response.Status.CREATED.getStatusCode()); 
    final BroadcastReceiver receiver = new BroadcastReceiver() { 
      @Override 
      public void onReceive(Context context, Intent intent) { 
        if (getResultCode() != Activity.RESULT_OK) { 
          statusCode.set(Response.Status. 
           INTERNAL_SERVER_ERROR.getStatusCode()); 
        } 
        sentLatch.countDown(); 
      } 
    }; 
    deskDroidService.registerReceiver(receiver,  
      new IntentFilter("com.steeplesoft.deskdroid.SMS_SENT")); 
    ArrayList<PendingIntent> sentPIs = new ArrayList<>(); 
    for (int i = 0; i < parts.size(); i++) { 
      sentPIs.add(PendingIntent.getBroadcast( 
        deskDroidService.getApplicationContext(), 0, 
        new Intent("com.steeplesoft.deskdroid.SMS_SENT"), 0)); 
    } 
    sms.sendMultipartTextMessage(message.getAddress(), null,
    parts, sentPIs, null); 
    sentLatch.await(5, TimeUnit.SECONDS); 
    deskDroidService.unregisterReceiver(receiver); 
    return Response.status(statusCode.get()).build(); 

```

为了确保我们已经收到了每个消息块的`Intent`，我们首先创建一个与消息中块的数量匹配的`CountDownLatch`。我们还创建一个`AtomicInteger`来保存状态码。我们这样做的原因是我们需要一个最终的变量，我们可以从我们的`BroadcastReceiver`中访问，但我们也需要能够更改这个值。`AtomicInteger`允许我们这样做。

我们创建并注册一个`BroadcastReceiver`，它分析`Intent`上的结果代码。如果不是`Activity.RESULT_OK`，我们将`statusCode`设置为`INTERNAL_SERVER_ERROR`。无论哪种方式，我们都会倒计时。

准备好我们的接收器后，我们创建一个`PendingIntent`的`List`，每个块一个，然后我们将它与我们的消息块列表一起传递给`SmsManager.sendMultipartTextMessage()`。消息发送是异步的，所以我们调用`sentLatch.await()`等待结果返回。我们将等待时间限制为五秒，以免永远等待。一旦等待时间到期或 latch 被清除，我们取消注册我们的接收器并返回状态码。

# 获取更新

到目前为止，我们可以看到所有的对话，查看对话中的单个消息，并发送新消息。但是，我们还不能在设备上收到新消息时获取更新，所以让我们从这次开始实现，从服务器部分开始。

为了获得一系列事件，我们将使用一个名为 Server-Sent Events 的功能，这是 W3C 规范，用于从服务器接收推送通知。我们通过在客户端和服务器设置步骤中注册`SseFeature`来启用 Jersey 中的此功能。要创建一个 SSE 端点，我们指定该方法返回媒体类型`SERVER_SENT_EVENTS`，并将`EventOutput`作为有效载荷返回：

```java
    @GET 
    @Path("status") 
    @Produces(SseFeature.SERVER_SENT_EVENTS) 
    @Secure 
    public EventOutput streamStatus() { 
      final EventOutput eventOutput = new EventOutput(); 
      // ... 
      return eventOutput; 
    } 

```

从 Jersey 文档中，我们了解到：

在方法返回 eventOutput 之后，Jersey 运行时会识别这是一个 ChunkedOutput 扩展，并不会立即关闭客户端连接。相反，它会将 HTTP 头写入响应流，并等待发送更多的块（SSE 事件）。此时，客户端可以读取头并开始监听各个事件。

然后，服务器会保持与客户端的套接字打开，并向其推送数据。但数据从哪里来呢？服务器发送事件端点创建一个“线程”，该线程将数据写入我们之前创建的`EventOutput`实例。当“线程”完成时，它调用`eventOutput.close()`，这表示运行时可以关闭客户端连接。为了流式更新，我们的“线程”如下所示：

```java
    final Thread thread = new Thread() { 
      @Override 
      public void run() { 
        final LinkedBlockingQueue<SmsMessage> queue =  
          new LinkedBlockingQueue<>(); 
        BroadcastReceiver receiver = null; 
        try { 
          receiver = new BroadcastReceiver() { 
            @Override 
            public void onReceive(Context context,  
             Intent intent) { 
               Bundle intentExtras = intent.getExtras(); 
               if (intentExtras != null) { 
                 Object[] sms = (Object[])  
                  intentExtras.get("pdus"); 
                 for (int i = 0; i < sms.length; ++i) { 
                   SmsMessage smsMessage =  
                     SmsMessage.createFromPdu( 
                       (byte[]) sms[i]); 
                       queue.add(smsMessage); 
                 } 
               } 
            } 
          }; 
          deskDroidService.registerReceiver(receiver, 
           new IntentFilter( 
             "android.provider.Telephony.SMS_RECEIVED")); 
          while (!eventOutput.isClosed()) { 
            SmsMessage message = queue.poll(5,  
             TimeUnit.SECONDS); 
            while (message != null) { 
              JSONObject json = new JSONObject() 
               .put("participant", message. 
                getDisplayOriginatingAddress()) 
               .put("body", message. 
                getDisplayMessageBody()); 
              eventOutput.write(new OutboundEvent.Builder() 
               .name("new-message") 
               .data(json.toString()) 
               .build() 
              ); 
              message = queue.poll(); 
            } 
          } 
        } catch (JSONException | InterruptedException |  
           IOException e) { 
          } finally { 
              try { 
                if (receiver != null) { 
                  deskDroidService.unregisterReceiver(receiver); 
                } 
                eventOutput.close(); 
              } catch (IOException ioClose) { 
                  // ... 
                } 
            } 
      } 
    }; 
    thread.setDaemon(true); 
    thread.start(); 

```

正如我们以前所见，我们设置了一个`BroadcastReceiver`，我们在这里注册并在“线程”结束之前注销，但这次，我们正在监听广播，即已收到短信消息。为了确保我们的“线程”不会陷入一个小的、紧凑的、快速的循环中，这将迅速耗尽设备的电池，我们使用了`LinkedBlockingQueue`。当收到消息时，我们从`Intent`中提取`SmsMessage`，并将它们添加到`queue`中。在我们的 while 循环中，我们尝试从`queue`中`take()`一个项目。如果我们找到一个，我们处理它以及可能在我们处理时已经在队列中或在我们处理时添加的任何其他项目。一旦`queue`为空，我们就会回到等待状态。我们对`take()`设置了超时，以确保线程可以响应退出条件，尤其是客户端断开连接。只要客户端保持连接，这将继续运行。接下来，让我们看看客户端。

我们将细节封装在`ConversationService.subscribeToNewMessageEvents()`中，如下所示：

```java
    public void subscribeToNewMessageEvents( 
      Consumer<Message> callback) { 
        Thread thread = new Thread() { 
          @Override 
          public void run() { 
            stopListening = false; 
            EventInput eventInput = getWebTarget().path("status") 
             .request() 
             .header(HttpHeaders.AUTHORIZATION,  
              getAuthorizationHeader()) 
               .get(EventInput.class); 
            while (!eventInput.isClosed() && !stopListening) { 
              final InboundEvent inboundEvent =  
                eventInput.read(); 
              if (inboundEvent == null) { 
                // connection has been closed 
                break; 
              } 
              if ("new-message".equals(inboundEvent.getName())){ 
                Message message =  
                  inboundEvent.readData(Message.class); 
                if (message != null) { 
                  callback.accept(message); 
                } 
              } 
            } 
          } 
        }; 
        thread.setDaemon(true); 
        thread.start(); 
    } 

```

在上述代码中，我们创建一个“线程”，在其中调用 SSE 端点。客户端的返回类型是`EventInput`。我们循环处理每个传入事件，我们将其作为`InboundEvent`获取。如果它为空，则连接已关闭，因此我们退出处理循环。如果它不为空，我们确保事件名称与我们正在等待的内容匹配--`new-message`。如果找到，我们提取事件有效载荷，即“消息”，并调用我们的回调，我们将其作为`Consumer<Message>`传递。

从应用程序中，我们以这种方式订阅状态流：

```java
    cs.subscribeToNewMessageEvents(this::handleMessageReceived); 

```

`handleMessageReceived()`看起来像这样：

```java
    protected void handleMessageReceived(final Message message) { 
      Platform.runLater(() -> { 
        Optional<Conversation> optional = conversations.stream() 
          .filter(c -> Objects.equal(c.getParticipant(),  
           message.getAddress())) 
          .findFirst(); 
        if (optional.isPresent()) { 
          Conversation c = optional.get(); 
          c.getMessages().add(message); 
          c.setSnippet(message.getBody()); 
          convList.refresh(); 
          if (c == conversation.get()) { 
            messages.setAll(c.getMessages()); 
            messageList.scrollTo(messages.size() - 1); 
          } 
        } else { 
            Conversation newConv = new Conversation(); 
            newConv.setParticipant(message.getAddress()); 
            newConv.setSnippet(message.getBody()); 
            newConv.setMessages(Arrays.asList(message)); 
            conversations.add(0, newConv); 
        } 
        final Taskbar taskbar = Taskbar.getTaskbar(); 
        if (taskbar.isSupported(Taskbar.Feature.USER_ATTENTION)) { 
          taskbar.requestUserAttention(true, false); 
        } 
        Toolkit.getDefaultToolkit().beep(); 
      }); 
    } 

```

处理这条新消息的第一步非常重要--我们将一个`Runnable`传递给`Platform.runLater()`。如果我们不这样做，任何修改用户界面的尝试都将失败。你已经被警告了。在我们的`Runnable`中，我们创建一个`Conversation`的`Stream`，对其进行`filter()`，寻找与“消息”发送者匹配的`Conversation`，然后抓取第一个（也是唯一的）匹配项。

如果我们在列表中找到“对话”，我们将这条新的“消息”添加到其列表中，并更新摘要（即“对话”的最后一条消息正文）。我们还要求“对话”列表“刷新（）”自身，以确保用户界面反映这些更改。最后，如果“对话”是当前选定的对话，我们会更新消息列表并滚动到底部，以确保新消息显示出来。

如果我们在列表中找不到“对话”，我们会创建一个新的对话，并将其添加到`ConversationObservable`中，这将导致`List`在屏幕上自动更新。

最后，我们尝试一些桌面集成任务。如果`Taskbar`支持`USER_ATTENTION`功能，我们请求用户注意。从 Javadocs 中我们了解到，*根据平台的不同，这可能通过任务区域中的图标弹跳或闪烁来进行视觉指示*。无论如何，我们都会发出蜂鸣声来引起用户的注意。

# 安全

还有一个我们还没有讨论的重要部分，那就是安全性。目前，任何拥有桌面应用程序的人理论上都可以连接到您的手机，查看您的消息，发送其他消息等。让我们现在来解决这个问题。

# 保护端点

为了保护 REST 服务器，我们将使用一个过滤器，就像我们在第九章中使用的那样，*使用 Monumentum 做笔记*。我们将首先定义一个注解，指定需要保护的端点，如下所示：

```java
    @NameBinding 
    @Retention(RetentionPolicy.RUNTIME) 
    @Target({ElementType.TYPE, ElementType.METHOD}) 
    public @interface Secure {} 

```

我们将把上述注解应用到每个受保护的端点上（为简洁起见，将注解压缩为一行）：

```java
    @GET @Path("conversations") @Secure 
    public Response getConversations() { 
      ... 
      @POST @Path("conversations") @Secure 
      public Response sendMessage(Message message)  
       throws InterruptedException { 
         ... 
         @GET @Path("status") @Produces(SseFeature.SERVER_SENT_EVENTS)  
         @Secure 
         public EventOutput streamStatus() { 
           ... 
           @GET @Path("participants/{address}") @Secure 
           public Response getParticipant( 
             @PathParam("address") String address) { 
               ... 

```

我们还需要一个过滤器来强制执行安全性，我们将其添加如下：

```java
    @Provider 
    @Secure 
    @Priority(Priorities.AUTHENTICATION) 
    public class SecureFilter implements ContainerRequestFilter { 
      private DeskDroidService deskDroidService; 

      public SecureFilter(DeskDroidService deskDroidService) { 
        this.deskDroidService = deskDroidService; 
      } 

      @Override 
      public void filter(ContainerRequestContext requestContext)  
        throws IOException { 
          try { 
            String authorizationHeader = requestContext. 
             getHeaderString(HttpHeaders.AUTHORIZATION); 
            String token = authorizationHeader. 
             substring("Bearer".length()).trim(); 
            final Key key = KeyGenerator. 
             getKey(deskDroidService.getApplicationContext()); 
            final JwtParser jwtParser =  
              Jwts.parser().setSigningKey(key); 
            jwtParser.parseClaimsJws(token); 
          } catch (Exception e) { 
              requestContext.abortWith(Response.status( 
                Response.Status.UNAUTHORIZED).build()); 
            } 
      } 
    } 

```

就像在第九章中一样，*使用 Monumentum 做笔记*，我们将使用**JSON Web Tokens**（**JWT**）来帮助验证和授权客户端。在这个过滤器中，我们从请求头中提取 JWT，并通过以下步骤验证它：

1.  从`KeyGenerator`获取签名密钥。

1.  使用签名密钥创建`JwtParser`。

1.  解析 JWT 中的声明。在这里，基本上只是对令牌本身进行验证。

1.  如果令牌无效，使用`UNAUTHORIZED`（`401`）中止请求。

`KeyGenerator`本身看起来有点像我们在第九章中看到的，*使用 Monumentum 做笔记*，但已经修改为以这种方式使用 Android API：

```java
    public class KeyGenerator { 
      private static Key key; 
      private static final Object lock = new Object(); 

      public static Key getKey(Context context) { 
        synchronized (lock) { 
          if (key == null) { 
            SharedPreferences sharedPref =  
              context.getSharedPreferences( 
                context.getString( 
                  R.string.preference_deskdroid),  
                   Context.MODE_PRIVATE); 
                  String signingKey = sharedPref.getString( 
                    context.getString( 
                      R.string.preference_signing_key), null); 
                  if (signingKey == null) { 
                    signingKey = UUID.randomUUID().toString(); 
                    final SharedPreferences.Editor edit =  
                      sharedPref.edit(); 
                    edit.putString(context.getString( 
                      R.string.preference_signing_key), 
                       signingKey); 
                    edit.commit(); 
                  } 
                  key = new SecretKeySpec(signingKey.getBytes(),
                   0, signingKey.getBytes().length, "DES"); 
          } 
        } 

        return key; 
      } 
    } 

```

由于我们可能同时从多个客户端接收请求，我们需要小心生成密钥。为了确保它只执行一次，我们将使用与服务器启动中看到的相同类型的同步/锁定。

一旦我们获得了锁，我们执行一个空检查，看看进程是否已经生成（或读取）了密钥。如果没有，我们就从`SharedPreferences`中读取签名密钥。如果它为空，我们就创建一个随机字符串（这里只是一个 UUID），并保存到`SharedPreferences`中以便下次重用。请注意，要保存到 Android 首选项，我们必须获得`SharedPreferences.Editor`的实例，写入字符串，然后`commit()`。一旦我们有了签名密钥，我们就创建实际的`SecretKeySpec`，用于签名和验证我们的 JWT。

# 处理授权请求

现在我们的端点已经得到保护，我们需要一种方式让客户端请求授权。为此，我们将公开一个新的端点，当然是不安全的，如下所示：

```java
    @POST 
    @Path("authorize") 
    @Consumes(MediaType.TEXT_PLAIN) 
    public Response getAuthorization(String clientCode) { 
      if (clientCode != null &&  
        clientCode.equals(deskDroidService.code)) { 
          String jwt = Jwts.builder() 
           .setSubject("DeskDroid") 
           .signWith(SignatureAlgorithm.HS512, 
            KeyGenerator.getKey( 
              deskDroidService.getApplicationContext())) 
               .compact(); 
          LocalBroadcastManager.getInstance( 
            deskDroidService.getApplicationContext()) 
           .sendBroadcast(new Intent( 
               DeskDroidService.CODE_ACCEPTED)); 
        return Response.ok(jwt).build(); 
      } 
      return Response.status(Response.Status.UNAUTHORIZED).build(); 
    } 

```

我们不需要一个更复杂的授权系统，可能需要用户名和密码或 OAuth2 提供者，我们将实现一个只需要一个随机数的简单系统：

1.  在手机上，用户请求添加一个新的客户端，并呈现一个随机数。

1.  在桌面应用程序中，用户输入数字，然后桌面应用程序将其 POST 到服务器。

1.  如果数字匹配，客户端将获得一个 JWT，它将在每个请求中发送。

1.  每次验证 JWT 以确保客户端被授权访问目标资源。

在这种方法中，我们获取客户端 POST 的数字（我们让 JAX-RS 从请求体中提取），然后将其与手机上生成的数字进行比较。如果它们匹配，我们创建 JWT，并将其返回给客户端。在这样做之前，我们广播一个带有动作`CODE_ACCEPTED`的意图。

这个数字是从哪里来的，为什么我们要广播这个意图？我们还没有详细研究过这个问题，但在主布局`activity_main.xml`中，有一个`FloatingActionButton`。我们将一个`onClick`监听器附加到这个上面，如下所示：

```java
    FloatingActionButton fab =  
      (FloatingActionButton) findViewById(R.id.fab); 
    fab.setOnClickListener(new View.OnClickListener() { 
      @Override 
      public void onClick(View view) { 
        startActivityForResult(new Intent( 
          getApplicationContext(),  
          AuthorizeClientActivity.class), 1); 
      } 
    }); 

```

当用户点击按钮时，将显示以下屏幕：

![](img/615b1a73-0b78-4a8f-b0a5-bab464247a9f.png)

客户端将使用这些信息进行连接和授权。`Activity`本身非常基本。它需要呈现 IP 地址和代码，然后响应客户端连接。所有这些都在我们的新`AuthorizeClientActivity`类的`onCreate()`中完成。我们从`WifiManager`中获取 IP 地址：

```java
    WifiManager wifiMgr = (WifiManager) getApplicationContext(). 
     getSystemService(WIFI_SERVICE); 
    String ipAddress = Formatter.formatIpAddress(wifiMgr. 
     getConnectionInfo().getIpAddress()); 

```

请记住，我们要求客户端连接到 Wi-Fi 网络。代码只是一个随机的 6 位数字：

```java
    String code = Integer.toString(100000 +  
     new Random().nextInt(900000)); 

```

监听我们之前看到的`Intent`，这表明客户端已经通过身份验证（很可能是在`Activity`显示后不久），我们注册另一个接收器如下：

```java
    messageReceiver = new BroadcastReceiver() { 
      @Override 
      public void onReceive(Context context, Intent intent) { 
        clientAuthenticated(); 
      } 
    }; 
    LocalBroadcastManager.getInstance(this).registerReceiver( 
      messageReceiver, new IntentFilter( 
        DeskDroidService.CODE_ACCEPTED)); 

```

我们还需要告诉`Service`这个新代码是什么，以便它可以验证它。为此，我们广播一个`Intent`如下：

```java
    Intent intent = new Intent(DeskDroidService.CODE_GENERATED); 
    intent.putExtra("code", code); 
    LocalBroadcastManager.getInstance(this).sendBroadcast(intent); 

```

我们已经在`DeskDroidService.onStartCommand()`中看到了广播的另一半，在那里从`Intent`中检索代码，并将其存储在服务中供`DeskDroidResource.getAuthorization()`使用。

最后，处理身份验证通知的这个方法只是清理接收器并关闭`Activity`：

```java
    protected void clientAuthenticated() { 
      LocalBroadcastManager.getInstance(this). 
        unregisterReceiver(messageReceiver); 
      setResult(2, new Intent()); 
      finish(); 
    } 

```

有了这个，当客户端连接并成功验证时，`Activity`关闭，用户返回到主`Activity`。

# 授权客户端

直到这一点，一切都假定桌面已经连接到手机。我们现在已经有足够的部件，可以有意义地讨论这个问题。

在应用程序的主`Menu`中，我们有两个`MenuItem`：`连接到手机`和`断开手机连接`。`连接到手机`处理程序如下所示：

```java
    @FXML 
    protected void connectToPhone(ActionEvent event) { 
      ConnectToPhoneController.showAndWait(); 
      if (!preferences.getToken().isEmpty()) { 
        refreshAndListen(); 
      } 
    } 

```

我们将使用现在熟悉的`showAndWait()`模式来显示模态对话框，并使用新的`ConnectToPhoneController`获取响应。用户界面非常简单，如下截图所示：

![](img/c46711ea-77d2-42dd-85fe-baa05d041061.png)

当用户点击确定时，我们将地址和代码保存在应用程序的首选项中，然后尝试对服务器进行授权，如下所示：

```java
    @FXML 
    public void connectToPhone(ActionEvent event) { 
      String address = phoneAddress.getText(); 
      String code = securityCode.getText(); 
      preferences.setPhoneAddress(address); 
      final ConversationService conversationService =  
        ConversationService.getInstance(); 

      conversationService.setPhoneAddress(address); 
      Optional<String> token = conversationService 
        .getAuthorization(code); 
      if (token.isPresent()) { 
        preferences.setToken(token.get()); 
        closeDialog(event); 
      } 
    } 

```

请注意，`Optional<String>`被用作`ConversationService.getAuthorization()`的返回类型。正如我们之前讨论过的，使用`Optional`可以更安全地处理可能为`null`的值。在这种情况下，如果`Optional`有一个值，那么我们已经成功验证。因此，我们将令牌保存到首选项中，并关闭对话框。

实际的身份验证由`ConversationService`处理：

```java
    public Optional<String> getAuthorization(String code) { 
      Response response = getWebTarget().path("authorize") 
       .request(MediaType.APPLICATION_JSON) 
       .post(Entity.text(code)); 
      Optional<String> result; 
      if(response.getStatus()==Response.Status.OK.getStatusCode()) { 
        token = response.readEntity(String.class); 
        result = Optional.of(token); 
      } else { 
          result = Optional.empty(); 
      } 
      return result; 
    } 

```

这个最后的方法通过`POST`将代码发送到服务器，如果状态码是`200`，我们将创建一个带有返回令牌的`Optional`。否则，我们返回一个空的`Optional`。

# 总结

在本章中，我们构建了一种不同类型的项目。我们有在 Android 上运行的应用程序，也有在桌面上运行的应用程序。然而，这个项目同时在两个平台上运行。一个没有另一个是不行的。这要求我们以稍微不同的方式构建东西，以确保两者同步。虽然有各种各样的方法可以解决这个问题，但我们选择在手机上使用 REST 服务器，桌面作为 REST 客户端。

在本章结束时，我们构建了一个安卓应用程序，它不仅提供了用户界面，还提供了一个后台进程（称为“服务”），并使用 Jersey 及其 Java SE 部署选项在安卓应用程序中嵌入了我们的 REST 服务器。您还学会了如何在安卓上使用系统提供的内容提供程序和平台 API 与文本（短信）进行交互，并使用服务器发送事件将这些消息流式传输到客户端。我们演示了如何在安卓中使用“意图”、“广播”和“广播接收器”在进程/线程之间发送消息。最后，在桌面端，我们构建了一个 JavaFX 客户端来显示和发送文本消息，它通过 Jersey REST 客户端连接到手机上的 REST 服务器，并消费了服务器发送的事件流，根据需要更新用户界面。

由于涉及的各个部分都很复杂，这可能是我们项目中最复杂的一个。这无疑是一个很好的方式来完成我们的项目列表。在下一章中，我们将看看 Java 的未来发展，以及一些其他可能值得关注的技术。
