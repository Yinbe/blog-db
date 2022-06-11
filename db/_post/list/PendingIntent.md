##### **PendingIntent作用**

PendingIntent**即将发生**的**意图**，它不是立刻执行某个行为，而是满足某些条件或触发某些事件后才执行指定的行为

PendingIntent的示例

```java
Intent intent = new Intent(this, SomeActivity.class);
 
// Creating a pending intent and wrapping our intent
PendingIntent pendingIntent = PendingIntent.getActivity(this, 1, intent, PendingIntent.FLAG_UPDATE_CURRENT);
try {
    // Perform the operation associated with our pendingIntent
    pendingIntent.send();
} catch (PendingIntent.CanceledException e) {
    e.printStackTrace();
}
```

##### 参数及说明

> **this** (context) :这是**PendingIntent** 在其中启动活动的上下文
>
> **requestCode：“ 1”是在以上示例中使用的发件人的私有请求代码。 以后再次以相同的方法使用它会返回相同的未决意图。 然后，我们可以执行各种操作，例如使用cancel（）取消待处理的意图等。**
>
> **intent：要启动的活动的明确意图对象**
>
> **flag** ：在上面的示例中使用的PendingIntent标志之一是**FLAG_UPDATE_CURRENT** 。 这说明如果先前的PendingIntent已经存在，那么当前的PendingIntent将使用最新的Intent更新它。 还有许多其他标志，例如**FLAG_CANCEL_CURRENT** 等。
>
>> 前面三个参数共同标志一个行为的唯一性，而第四个参数flags：
>>
>> FLAG_CANCEL_CURRENT：如果当前系统中已经存在一个相同的PendingIntent对象，那么就将先将已有的PendingIntent取消，然后重新生成一个PendingIntent对象。
>> FLAG_NO_CREATE：如果当前系统中不存在相同的PendingIntent对象，系统将不会创建该PendingIntent对象而是直接返回null，如果之前设置过，这次就能获取到。
>> FLAG_ONE_SHOT：该PendingIntent只作用一次。在该PendingIntent对象通过send()方法触发过后，PendingIntent将自动调用cancel()进行销毁，那么如果你再调用send()方法的话，系统将会返回一个SendIntentException。
>> FLAG_UPDATE_CURRENT：如果系统中有一个和你描述的PendingIntent对等的PendingInent，那么系统将使用该PendingIntent对象，但是会使用新的Intent来更新之前PendingIntent中的Intent对象数据，例如更新Intent中的Extras
>>

PendingIntent的使用场景主要用于闹钟、通知、桌面部件

A应用希望让B应用帮忙触发一个行为，这是跨应用的通信，需要 Android 系统作为中间人，这里的中间人就是 ActivityManager。 A应用创建建 PendingIntent，在创建 PendingIntent 的过程中，向 ActivityManager 注册了这个 PendingIntent，所以，即使A应用死了，当它再次苏醒时，只要提供相同的参数，还是可以获取到之前那个 PendingIntent 的。当 A 将 PendingIntent 调用系统 API 比如 AlarmManager.set()，实际是将权限给了B应用，这时候， B应用可以根据参数信息，来从 ActivityManager 获取到 A 设置的 PendingIntent

Notification实例

```java
public void sendNotification() {
 
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
        builder.setSmallIcon(android.R.drawable.ic_dialog_alert);
        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://www.journaldev.com/"));
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
        builder.setContentIntent(pendingIntent);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
        builder.setContentTitle("Notifications Title");
        builder.setContentText("Your notification content here.");
        builder.setSubText("Tap to view the website.");
 
        NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
 
        // Will display the notification in the notification bar
        notificationManager.notify(1, builder.build());
    }

```

这里创建了打开网页 百度 的intent，并给到了通知栏。当点击通知栏时，就会触发这个intent，打开百度。

```java
	Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://www.baidu.com/"));  
	PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
    	builder.setContentIntent(pendingIntent);
```

AppWidgetProvider

```java
	@Override
	public void onUpdate(Context context, AppWidgetManager appWidgetManager,
			int[] appWidgetIds) {
		System.out.println("点击反应");
		//创建一个Intent对象
		Intent intent=new Intent();
		//为Intent对象设置Action
		intent.setAction(UPDATE_ACTION);
		//使用getBroadcast方法，得到一个PendingIntent对象，当该对象执行时，会发送一个广播
		PendingIntent pendingIntent=PendingIntent.getBroadcast(context, 0, intent, 0);
		RemoteViews remoteViews=new RemoteViews(context.getPackageName(), R.layout.appwidget_layout);
		remoteViews.setOnClickPendingIntent(R.id.bId, pendingIntent);
		appWidgetManager.updateAppWidget(appWidgetIds,remoteViews);
	}
```

相似的类 `IntentSender`

对 Intent 和要对其执行的目标操作的描述。 返回的对象可以交给其他应用程序，以便它们稍后可以代表您执行您描述的操作。
通过将 IntentSender 提供给另一个应用程序，您授予它执行您指定的操作的权利，就好像另一个应用程序是您自己一样（具有相同的权限和身份）。 因此，您应该注意如何构建 IntentSender：例如，通常，您提供的基本 Intent 会将组件名称显式设置为您自己的组件之一，以确保它最终被发送到那里而不是其他地方。
IntentSender 本身只是对系统维护的令牌的引用，该令牌描述用于检索它的原始数据。 这意味着，即使拥有它的应用程序的进程被终止，IntentSender 本身仍可用于其他已赋予它的进程。 如果创建的应用程序稍后重新检索相同类型的 IntentSender（相同的操作、相同的 Intent 操作、数据、类别和组件以及相同的标志），如果它仍然有效，它将收到一个表示相同令牌的 IntentSender。此类的实例不能直接创建，而必须从现有的android.app.PendingIntent使用PendingIntent.getIntentSender() 。

Activity有个startIntentSenderForResult方法

```java
    public static void startIntentSenderForResult(@NonNull Activity activity,
            @NonNull IntentSender intent, int requestCode, @Nullable Intent fillInIntent,
            int flagsMask, int flagsValues, int extraFlags, @Nullable Bundle options)
            throws IntentSender.SendIntentException {
        if (Build.VERSION.SDK_INT >= 16) {
            activity.startIntentSenderForResult(intent, requestCode, fillInIntent, flagsMask,
                    flagsValues, extraFlags, options);
        } else {
            activity.startIntentSenderForResult(intent, requestCode, fillInIntent, flagsMask,
                    flagsValues, extraFlags);
        }
    }
```
