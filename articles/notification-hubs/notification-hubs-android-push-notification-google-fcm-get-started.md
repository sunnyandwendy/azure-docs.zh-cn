<properties
	pageTitle="使用 Azure 通知中心和 Firebase Cloud Messaging 将推送通知发送到 Android | Azure"
	description="本教程介绍了如何使用 Azure 通知中心和 Firebase Cloud Messaging 将通知推送到 Android 设备。"
	services="notification-hubs"
	documentationCenter="android"
	keywords="推送通知, 推送通知, android 推送通知, fcm, firebase cloud messaging"
	authors="wesmc7777"
	manager="erikre"
	editor=""/>
<tags
	ms.service="notification-hubs"
	ms.date="07/14/2016"
	wacn.date=""/>

# 通过 Azure 通知中心向 Android 发送推送通知

[AZURE.INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

##概述

> [AZURE.IMPORTANT] 本主题演示了使用 Google Firebase Cloud Messaging (FCM) 的推送通知。如果您仍在使用 Google Cloud Messaging (GCM)，请参阅 [Sending push notifications to Android with Azure Notification Hubs and GCM](/documentation/articles/notification-hubs-android-push-notification-google-gcm-get-started/)（使用 Azure 通知中心和 GCM 将推送通知发送到 Android）。

本教程介绍了如何使用 Azure 通知中心和 Firebase Cloud Messaging 将推送通知发送到 Android 应用程序。您将创建一个空白 Android 应用，它使用 Firebase Cloud Messaging (FCM) 接收推送通知。



[AZURE.INCLUDE [notification-hubs-hero-slug](../../includes/notification-hubs-hero-slug.md)]

可以从[此处](https://github.com/Azure/azure-notificationhubs-samples/tree/master/Android/GetStartedFirebase)的 GitHub 中下载本教程的已完成代码。


##先决条件

> [AZURE.IMPORTANT] 若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](/pricing/1rmb-trial-full/?form-type=identityauth)。

- 除了需要上面提到的可用 Azure 帐户外，本教程还要求具有最新版本的 [Android Studio](http://go.microsoft.com/fwlink/?LinkId=389797)。
- 适用于 Firebase Cloud Messaging 的 Android 2.3 或更高版本。
- Firebase Cloud Messaging 要求安装 Google Repository 修订版 27 或更高版本。
- 适用于 Firebase Cloud Messaging 的 Google Play Services 9.0.2 或更高版本。
- 完成本教程是学习有关 Android 应用的所有其他通知中心教程的先决条件。


##新建 Android Studio 项目

1. 在 Android Studio 中，启动新的 Android Studio 项目。

   	![Android Studio - 新项目](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-android-studio-new-project.png)

2. 选择“手机和平板电脑”外形规格和你要支持的“最低 SDK 版本”。然后单击“下一步”。

   	![Android Studio - 项目创建工作流](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-android-studio-choose-form-factor.png)

3. 选择“空活动”作为主活动，单击“下一步”，然后单击“完成”。


##创建支持 Google Cloud Messaging 的项目

[AZURE.INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging.md)]


##配置新通知中心

[AZURE.INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

&emsp;&emsp;6.在通知中心的“设置”边栏选项卡中，选择“通知服务”，然后选择“Google (GCM)”。输入先前从 [Firebase 控制台](https://firebase.google.com/console/) 复制的 FCM 服务器密钥，然后单击“保存”。

&emsp;&emsp;![Azure 通知中心 - Google (GCM)](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-gcm-api.png)

现在您的通知中心已配置为使用 Firebase Cloud Messagin，并且您具有用于注册您的应用以收发推送通知的连接字符串。

##<a id="connecting-app"></a>将你的应用连接到通知中心

###将 Google Play 服务添加到项目

[AZURE.INCLUDE [添加 Play Services](../../includes/notification-hubs-android-studio-add-google-play-services.md)]

###添加 Azure 通知中心库


1. 在**应用**的 `Build.Gradle` 文件的 **dependencies** 节中添加以下行。

		compile 'com.microsoft.azure:notification-hubs-android-sdk:0.4@aar'
		compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'

2. 在 **dependencies** 节的后面添加以下存储库。

		repositories {
		    maven {
		        url "http://dl.bintray.com/microsoftazuremobile/SDK"
		    }
		}

### 更新 AndroidManifest.xml。


1. 若要支持 FCM，我们必须在代码中实现实例 ID 侦听器服务，以便使用 [Google 的 FirebaseInstanceId API](https://firebase.google.com/docs/reference/android/com/google/firebase/iid/FirebaseInstanceId) 来[获取注册令牌](https://firebase.google.com/docs/cloud-messaging/android/client#sample-register)。在本教程中，我们将该类命名为 `MyInstanceIDService`。
 
	将以下服务定义添加到 AndroidManifest.xml 文件的 `<application>` 标记内。

        <service android:name=".MyInstanceIDService">
            <intent-filter>
                <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
            </intent-filter>
        </service>



2. 从 FirebaseInstanceId API 收到 FCM 注册令牌后，我们将使用它[在 Azure 通知中心注册](/documentation/articles/notification-hubs-push-notification-registration-management/)。我们将使用名为 `RegistrationIntentService` 的 `IntentService` 在后台支持此注册。此服务还负责刷新 FCM 注册令牌。
 
	将以下服务定义添加到 AndroidManifest.xml 文件的 `<application>` 标记内。

        <service
            android:name=".RegistrationIntentService"
            android:exported="false">
        </service>



3. 我们还要定义通知接收者。将以下接收者定义添加到 AndroidManifest.xml 文件的 `<application>` 标记内。将 `<your package>` 占位符替换为 `AndroidManifest.xml` 文件顶部显示的实际包名称。

		<receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
		    android:permission="com.google.android.c2dm.permission.SEND">
		    <intent-filter>
		        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
		        <category android:name="<your package name>" />
		    </intent-filter>
		</receiver>



4. 在 `</application>` 标记下面添加以下必要的 FCM 相关权限。请确保将 `<your package>` 替换为 `AndroidManifest.xml` 文件顶部显示的包名称。

	有关这些权限的详细信息，请参阅[安装适用于 Android 的 GCM 客户端应用](https://developers.google.com/cloud-messaging/android/client#manifest)和[将适用于 Android 的 GCM 客户端应用迁移到 Firebase Cloud Messaging](https://developers.google.com/cloud-messaging/android/android-migrate-fcm#remove_the_permissions_required_by_gcm)。

		<uses-permission android:name="android.permission.INTERNET"/>
		<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
		<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />


### 添加代码


1. 在“项目”视图中，展开“应用”>“src”>“main”>“java”。右键单击 **java** 下的包文件夹，单击“新建”，然后单击“Java 类”。添加名为 `NotificationSettings` 的新类。

	![Android Studio - 新 Java 类](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hub-android-new-class.png)

	确保在 `NotificationSettings` 类的以下代码中更新这三个占位符：
	* **SenderId**：之前在 [Firebase 控制台](https://firebase.google.com/console/)中的项目设置的**Cloud Messaging** 选项卡中获取的发件人 ID。
	* **HubListenConnectionString**：中心的 **DefaultListenAccessSignature** 连接字符串。您可以复制此连接字符串，方法是在 [Azure 门户]上您的中心的“设置”边栏选项卡上单击“访问策略”。
	* **HubName**：使用 [Azure 门户]的中心边栏选项卡上显示的通知中心的名称。

	`NotificationSettings`代码：

		public class NotificationSettings {
		    public static String SenderId = "<Your project number>";
		    public static String HubName = "<Your HubName>";
		    public static String HubListenConnectionString = "<Enter your DefaultListenSharedAccessSignature connection string>";
		}

2. 使用上述步骤，添加另一个名为 `MyInstanceIDService` 的新类。这是我们的实例 ID 侦听器服务实现。

	此类的代码将调用 `IntentService` 以在后台[刷新 FCM 令牌](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)。

		import android.content.Intent;
		import android.util.Log;
		import com.google.firebase.iid.FirebaseInstanceIdService;
		
		
		public class MyInstanceIDService extends FirebaseInstanceIdService {
		
		    private static final String TAG = "MyInstanceIDService";
		
		    @Override
		    public void onTokenRefresh() {
		
		        Log.d(TAG, "Refreshing GCM Registration Token");
		
		        Intent intent = new Intent(this, RegistrationIntentService.class);
		        startService(intent);
		    }
		};


3. 将另一个名为 `RegistrationIntentService` 的新类添加到项目。这是我们的 `IntentService` 实现，用于处理[刷新 FCM 令牌](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens)和[在通知中心注册](/documentation/articles/notification-hubs-push-notification-registration-management/)。

	针对此类使用以下代码。

		import android.app.IntentService;
		import android.content.Intent;
		import android.content.SharedPreferences;
		import android.preference.PreferenceManager;
		import android.util.Log;		
		import com.google.firebase.iid.FirebaseInstanceId;
		import com.microsoft.windowsazure.messaging.NotificationHub;
		
		public class RegistrationIntentService extends IntentService {
		
		    private static final String TAG = "RegIntentService";
		
		    private NotificationHub hub;
		
		    public RegistrationIntentService() {
		        super(TAG);
		    }
		
		    @Override
		    protected void onHandleIntent(Intent intent) {
		
		        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
		        String resultString = null;
		        String regID = null;
		        String storedToken = null;
		
		        try {
		            String FCM_token = FirebaseInstanceId.getInstance().getToken();
		            Log.d(TAG, "FCM Registration Token: " + FCM_token);
		
		            // Storing the registration id that indicates whether the generated token has been
		            // sent to your server. If it is not stored, send the token to your server,
		            // otherwise your server should have already received the token.
		            if (((regID=sharedPreferences.getString("registrationID", null)) == null)){
		
		                NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
		                        NotificationSettings.HubListenConnectionString, this);
		                Log.d(TAG, "Attempting a new registration with NH using FCM token : " + FCM_token);
		                regID = hub.register(FCM_token).getRegistrationId();
		
		                // If you want to use tags...
		                // Refer to : /documentation/articles/notification-hubs-routing-tag-expressions/
		                // regID = hub.register(token, "tag1,tag2").getRegistrationId();
		
		                resultString = "New NH Registration Successfully - RegId : " + regID;
		                Log.d(TAG, resultString);
		
		                sharedPreferences.edit().putString("registrationID", regID ).apply();
		                sharedPreferences.edit().putString("FCMtoken", FCM_token ).apply();
		            }
		
		            // Check if the token may have been compromised and needs refreshing.
		            else if ((storedToken=sharedPreferences.getString("FCMtoken", "")) != FCM_token) {
		
		                NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
		                        NotificationSettings.HubListenConnectionString, this);
		                Log.d(TAG, "NH Registration refreshing with token : " + FCM_token);
		                regID = hub.register(FCM_token).getRegistrationId();
		
		                // If you want to use tags...
		                // Refer to : /documentation/articles/notification-hubs-routing-tag-expressions/
		                // regID = hub.register(token, "tag1,tag2").getRegistrationId();
		
		                resultString = "New NH Registration Successfully - RegId : " + regID;
		                Log.d(TAG, resultString);
		
		                sharedPreferences.edit().putString("registrationID", regID ).apply();
		                sharedPreferences.edit().putString("FCMtoken", FCM_token ).apply();
		            }
		
		            else {
		                resultString = "Previously Registered Successfully - RegId : " + regID;
		            }
		        } catch (Exception e) {
		            Log.e(TAG, resultString="Failed to complete registration", e);
		            // If an exception happens while fetching the new token or updating our registration data
		            // on a third-party server, this ensures that we'll attempt the update at a later time.
		        }
		
		        // Notify UI that registration has completed.
		        if (MainActivity.isVisible) {
		            MainActivity.mainActivity.ToastNotify(resultString);
		        }
		    }
		}


		
4. 在 `MainActivity` 类中，在类声明的上面添加以下 `import` 语句。

		import com.google.android.gms.common.ConnectionResult;
		import com.google.android.gms.common.GoogleApiAvailability;
		import com.microsoft.windowsazure.notifications.NotificationsManager;
		import android.content.Intent;
		import android.util.Log;
		import android.widget.TextView;
		import android.widget.Toast;

5. 将以下私有成员添加到类的顶部。我们将使用这些成员来[检查 Google 推荐的 Google Play 服务的可用性](https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk)。

	    public static MainActivity mainActivity;
    	public static Boolean isVisible = false;	
	    private static final String TAG = "MainActivity";
	    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;

6. 在 `MainActivity` 类中，添加以下方法来检查 Google Play 服务的可用性。

	    /**
	     * Check the device to make sure it has the Google Play Services APK. If
	     * it doesn't, display a dialog that allows users to download the APK from
	     * the Google Play Store or enable it in the device's system settings.
	     */
	    private boolean checkPlayServices() {
	        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
	        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
	        if (resultCode != ConnectionResult.SUCCESS) {
	            if (apiAvailability.isUserResolvableError(resultCode)) {
	                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST)
	                        .show();
	            } else {
	                Log.i(TAG, "This device is not supported by Google Play Services.");
	                ToastNotify("This device is not supported by Google Play Services.");
	                finish();
	            }
	            return false;
	        }
	        return true;
	    }


7. 在 `MainActivity` 类中添加以下代码，用于在调用 `IntentService` 之前检查 Google Play 服务，以获取 FCM 注册令牌并向通知中心注册。

	    public void registerWithNotificationHubs()
	    {
	        if (checkPlayServices()) {
	            // Start IntentService to register this application with FCM.
	            Intent intent = new Intent(this, RegistrationIntentService.class);
	            startService(intent);
	        }
	    }


8. 在 `MainActivity` 类的 `OnCreate` 方法中添加以下代码，以便在创建活动时开始注册过程。

	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        mainActivity = this;
	        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId, MyHandler.class);
	        registerWithNotificationHubs();
	    }


9. 将其他这些方法添加到 `MainActivity`，以验证和报告应用状态。

	    @Override
	    protected void onStart() {
	        super.onStart();
	        isVisible = true;
	    }
	
	    @Override
	    protected void onPause() {
	        super.onPause();
	        isVisible = false;
	    }
	
	    @Override
	    protected void onResume() {
	        super.onResume();
	        isVisible = true;
	    }
	
	    @Override
	    protected void onStop() {
	        super.onStop();
	        isVisible = false;
	    }
	
	    public void ToastNotify(final String notificationMessage) {
	        runOnUiThread(new Runnable() {
	            @Override
	            public void run() {
	                Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
	                TextView helloText = (TextView) findViewById(R.id.text_hello);
	                helloText.setText(notificationMessage);
	            }
	        });
	    }


10. `ToastNotify` 方法使用 *"Hello World"* `TextView` 控件持续报告应用状态和通知。在 activity\_main.xml 布局中，为该控件添加以下 ID。

        android:id="@+id/text_hello"

11. 接下来，我们将为 AndroidManifest.xml 中定义的接收者添加一个子类。将另一个名为 `MyHandler` 的新类添加到项目。

12. 在 `MyHandler.java` 的顶部添加以下 import 语句：

		import android.app.NotificationManager;
		import android.app.PendingIntent;
		import android.content.Context;
		import android.content.Intent;
		import android.media.RingtoneManager;
		import android.net.Uri;
		import android.os.Bundle;
		import android.support.v4.app.NotificationCompat;
		import com.microsoft.windowsazure.notifications.NotificationsHandler;

13. 为 `MyHandler` 类添加以下代码，使其成为 `com.microsoft.windowsazure.notifications.NotificationsHandler` 的子类。

	此代码将重写 `OnReceive` 方法，因此处理程序将报告所收到的通知。处理程序还使用 `sendNotification()` 方法将推送通知发送到 Android 通知管理器。应用未运行但收到通知时，应执行 `sendNotification()` 方法。

		public class MyHandler extends NotificationsHandler {
		    public static final int NOTIFICATION_ID = 1;
		    private NotificationManager mNotificationManager;
		    NotificationCompat.Builder builder;
		    Context ctx;
		
		    @Override
		    public void onReceive(Context context, Bundle bundle) {
		        ctx = context;
		        String nhMessage = bundle.getString("message");
		        sendNotification(nhMessage);
		        if (MainActivity.isVisible) {
		            MainActivity.mainActivity.ToastNotify(nhMessage);
		        }
		    }
		
		    private void sendNotification(String msg) {
		
		        Intent intent = new Intent(ctx, MainActivity.class);
		        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
		
		        mNotificationManager = (NotificationManager)
		                ctx.getSystemService(Context.NOTIFICATION_SERVICE);
		
		        PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
		                intent, PendingIntent.FLAG_ONE_SHOT);
		
		        Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
		        NotificationCompat.Builder mBuilder =
		                new NotificationCompat.Builder(ctx)
		                        .setSmallIcon(R.mipmap.ic_launcher)
		                        .setContentTitle("Notification Hub Demo")
		                        .setStyle(new NotificationCompat.BigTextStyle()
		                                .bigText(msg))
		                        .setSound(defaultSoundUri)
		                        .setContentText(msg);
		
		        mBuilder.setContentIntent(contentIntent);
		        mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
		    }
		}


14. 在 Android Studio 的菜单栏上，单击“生成”>“重新生成项目”，确保您的代码中没有任何错误。
15. 在设备上运行该应用，验证其是否已成功注册到通知中心。

	> [AZURE.NOTE] 除非已调用实例 ID 服务的 `onTokenRefresh()` 方法，否则最初启动时注册可能会失败。刷新即可成功注册到通知中心。

##发送推送通知

您可以通过在 [Azure 门户]中发送通知来测试您的应用接收推送通知的功能，如下所示，您可以在中心边栏选项卡中查找“疑难解答”部分来进行此测试。

![Azure 通知中心 - 测试发送](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-test-send.png)

[AZURE.INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

## （可选）直接从应用程序发送推送通知

>[AZURE.IMPORTANT] 提供这个从客户端应用发送通知查询的示例仅供学习。由于这需要将 `DefaultFullSharedAccessSignature` 呈现在客户端应用中，这使您的通知中心面临这样的风险：即用户可能会获得相应访问权限将未经授权的通知发送到您的客户端。

通常，你会使用后端服务器发送通知。在某些情况下，你可能希望能够直接从客户端应用程序发送推送通知。本部分说明了如何使用 [Azure 通知中心 REST API](https://msdn.microsoft.com/library/azure/dn223264.aspx) 从客户端发送通知。

1. 在 Android Studio 的项目视图中展开“应用”>“src”>“main”>“res”>“layout”。打开 `activity_main.xml` 布局文件，然后单击“文本”选项卡以更新此文件的文本内容。使用以下代码更新此文件，此代码将添加新的 `Button` 和 `EditText` 控件，用于将推送通知消息发送到通知中心。将此代码添加到底部紧靠 `</RelativeLayout>` 前面的位置。

	    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/send_button"
        android:id="@+id/sendbutton"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true"
        android:onClick="sendNotificationButtonOnClick" />

	    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editTextNotificationMessage"
        android:layout_above="@+id/sendbutton"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="42dp"
        android:hint="@string/notification_message_hint" />

2. 在 Android Studio 的项目视图中展开“应用”>“src”>“main”>“res”>“values”。打开 `strings.xml` 文件，并添加新的 `Button` 和 `EditText` 控件引用的字符串值。在文件底部紧靠在 `</resources>` 前面的位置添加这些值。

        <string name="send_button">Send Notification</string>
        <string name="notification_message_hint">Enter notification message text</string>


3. 在 `NotificationSetting.java` 文件中，将以下设置添加到 `NotificationSettings` 类。

	使用中心的 **DefaultFullSharedAccessSignature** 连接字符串更新 `HubFullAccess`。可从 [Azure 门户]复制此连接字符串，方法是单击通知中心的“设置”边栏选项卡上的“访问策略”。

		public static String HubFullAccess = "<Enter Your DefaultFullSharedAccessSignature Connection string>";

4. 在 `MainActivity.java` 文件中，将以下 `import` 语句添加在 `MainActivity` 类的上面。

		import java.io.BufferedOutputStream;
		import java.io.BufferedReader;
		import java.io.InputStreamReader;
		import java.io.OutputStream;
		import java.net.HttpURLConnection;
		import java.net.URL;
		import java.net.URLEncoder;
		import javax.crypto.Mac;
		import javax.crypto.spec.SecretKeySpec;
		import android.util.Base64;
		import android.view.View;
		import android.widget.EditText;

6. 在 `MainActivity.java` 文件中，在 `MainActivity` 类的最上面添加以下成员。

	    private String HubEndpoint = null;
	    private String HubSasKeyName = null;
	    private String HubSasKeyValue = null;

6. 你必须创建软件访问签名 (SaS) 令牌对 POST 请求进行身份验证，以便将消息发送到通知中心。为此，可以分析连接字符串中的密钥数据，然后按照[基本概念](http://msdn.microsoft.com/library/azure/dn495627.aspx) REST API 参考中所述创建 SaS 令牌。以下代码是示例实现。

	在 `MainActivity.java` 中，将以下方法添加到 `MainActivity` 类，以分析连接字符串。

	    /**
	     * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx
	     * to parse the connection string so a SaS authentication token can be
	     * constructed.
	     *
	     * @param connectionString This must be the DefaultFullSharedAccess connection
	     *                         string for this example.
	     */
	    private void ParseConnectionString(String connectionString)
	    {
	        String[] parts = connectionString.split(";");
	        if (parts.length != 3)
	            throw new RuntimeException("Error parsing connection string: "
	                    + connectionString);
	
	        for (int i = 0; i < parts.length; i++) {
	            if (parts[i].startsWith("Endpoint")) {
	                this.HubEndpoint = "https" + parts[i].substring(11);
	            } else if (parts[i].startsWith("SharedAccessKeyName")) {
	                this.HubSasKeyName = parts[i].substring(20);
	            } else if (parts[i].startsWith("SharedAccessKey")) {
	                this.HubSasKeyValue = parts[i].substring(16);
	            }
	        }
	    }


7. 在 `MainActivity.java` 中，将以下方法添加到 `MainActivity` 类，以创建 SaS 身份验证令牌。

	    /**
	     * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx to
	     * construct a SaS token from the access key to authenticate a request.
	     *
	     * @param uri The unencoded resource URI string for this operation. The resource
	     *            URI is the full URI of the Service Bus resource to which access is
	     *            claimed. For example,
	     *            "http://<namespace>.servicebus.windows.net/<hubName>"
	     */
	    private String generateSasToken(String uri) {
	
	        String targetUri;
	        String token = null;
	        try {
	            targetUri = URLEncoder
	                    .encode(uri.toString().toLowerCase(), "UTF-8")
	                    .toLowerCase();
	
	            long expiresOnDate = System.currentTimeMillis();
	            int expiresInMins = 60; // 1 hour
	            expiresOnDate += expiresInMins * 60 * 1000;
	            long expires = expiresOnDate / 1000;
	            String toSign = targetUri + "\n" + expires;
	
	            // Get an hmac_sha1 key from the raw key bytes
	            byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
	            SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");
	
	            // Get an hmac_sha1 Mac instance and initialize with the signing key
	            Mac mac = Mac.getInstance("HmacSHA256");
	            mac.init(signingKey);
	
	            // Compute the hmac on input data bytes
	            byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));
	
	            // Using android.util.Base64 for Android Studio instead of
	            // Apache commons codec
	            String signature = URLEncoder.encode(
	                    Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");
	
	            // Construct authorization string
	            token = "SharedAccessSignature sr=" + targetUri + "&sig="
	                    + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
	        } catch (Exception e) {
	            if (isVisible) {
	                ToastNotify("Exception Generating SaS : " + e.getMessage().toString());
	            }
	        }
	
	        return token;
	    }



8. 在 `MainActivity.java` 中，将以下方法添加到 `MainActivity` 类，以使用内置的 REST API 处理“发送通知”按钮的点击行为，并将推送通知消息发送到中心。

	    /**
	     * Send Notification button click handler. This method parses the
	     * DefaultFullSharedAccess connection string and generates a SaS token. The
	     * token is added to the Authorization header on the POST request to the
	     * notification hub. The text in the editTextNotificationMessage control
	     * is added as the JSON body for the request to add a GCM message to the hub.
	     *
	     * @param v
	     */
	    public void sendNotificationButtonOnClick(View v) {
	        EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
	        final String json = "{"data":{"message":"" + notificationText.getText().toString() + ""}}";
	
	        new Thread()
	        {
	            public void run()
	            {
	                try
	                {
	                    // Based on reference documentation...
	                    // http://msdn.microsoft.com/library/azure/dn223273.aspx
	                    ParseConnectionString(NotificationSettings.HubFullAccess);
	                    URL url = new URL(HubEndpoint + NotificationSettings.HubName +
	                            "/messages/?api-version=2015-01");
	
	                    HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();
	
	                    try {
	                        // POST request
	                        urlConnection.setDoOutput(true);
	
	                        // Authenticate the POST request with the SaS token
	                        urlConnection.setRequestProperty("Authorization", 
								generateSasToken(url.toString()));
	
	                        // Notification format should be GCM
	                        urlConnection.setRequestProperty("ServiceBusNotification-Format", "gcm");
	
	                        // Include any tags
	                        // Example below targets 3 specific tags
	                        // Refer to : https://azure.microsoft.com/en-us/documentation/articles/notification-hubs-routing-tag-expressions/
	                        // urlConnection.setRequestProperty("ServiceBusNotification-Tags", 
							//		"tag1 || tag2 || tag3");
	
	                        // Send notification message
	                        urlConnection.setFixedLengthStreamingMode(json.length());
	                        OutputStream bodyStream = new BufferedOutputStream(urlConnection.getOutputStream());
	                        bodyStream.write(json.getBytes());
	                        bodyStream.close();
	
	                        // Get reponse
	                        urlConnection.connect();
	                        int responseCode = urlConnection.getResponseCode();
	                        if ((responseCode != 200) && (responseCode != 201)) {
                                BufferedReader br = new BufferedReader(new InputStreamReader((urlConnection.getErrorStream())));
                                String line;
                                StringBuilder builder = new StringBuilder("Send Notification returned " +
                                        responseCode + " : ")  ;
                                while ((line = br.readLine()) != null) {
                                    builder.append(line);
                                }

                                ToastNotify(builder.toString());
	                        }
	                    } finally {
	                        urlConnection.disconnect();
	                    }
	                }
	                catch(Exception e)
	                {
	                    if (isVisible) {
	                        ToastNotify("Exception Sending Notification : " + e.getMessage().toString());
	                    }
	                }
	            }
	        }.start();
	    }



##测试应用程序

####模拟器中的推送通知

如果你想要在模拟器中测试推送通知，请确保模拟器映像支持你为应用程序选择的 Google API 级别。如果映像不支持本机 Google API，那么此测试最终将以 **SERVICE\_NOT\_AVAILABLE** 异常结束。

除上述条件外，请确保已将您的 Google 帐户添加到运行的模拟器的“设置”>“帐户”下。否则，尝试向 GCM 注册可能会导致 **AUTHENTICATION\_FAILED** 异常。

####运行应用程序

1. 运行应用，并留意报告注册成功的注册 ID。

   	![测试 Android - 通道注册](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-android-studio-registered.png)

2. 输入一条要发送到已在中心注册的所有 Android 设备的通知消息。

   	![测试 Android - 发送一条消息](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-android-studio-set-message.png)  


3. 按“发送通知”。运行此应用的所有设备将显示一个包含此推送通知消息的 `AlertDialog` 实例。未运行此应用程序，但之前已注册推送通知的设备将在 Android 通知管理器中收到通知。从左上角向下轻扫即可查看通知。

   	![测试 Android - 通知](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-android-studio-received-message.png)

##后续步骤

建议下一步学习[使用通知中心向用户推送通知]教程。它将显示如何使用标记从 ASP.NET 后端将通知发送到目标特定的用户。

如果要按兴趣组划分用户，可以查看 [Use Notification Hubs to send breaking news]（使用通知中心发送最新消息）教程。

若要了解有关通知中心的更多一般信息，请参阅 [Notification Hubs Guidance]（通知中心指南）。

<!-- Images. -->



<!-- URLs. -->
[Get started with push notifications in Mobile Services]: /documentation/articles/mobile-services-javascript-backend-android-get-started-push/
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: http://go.microsoft.com/fwlink/?LinkId=389800
[Azure Classic Portal]: https://manage.windowsazure.cn/
[Notification Hubs Guidance]: /documentation/articles/notification-hubs-push-notification-overview/
[使用通知中心向用户推送通知]: /documentation/articles/notification-hubs-aspnet-backend-android-notify-users/
[Use Notification Hubs to send breaking news]: /documentation/articles/notification-hubs-aspnet-backend-android-breaking-news/
[Azure 门户]: https://portal.azure.cn

<!---HONumber=Mooncake_0822_2016-->