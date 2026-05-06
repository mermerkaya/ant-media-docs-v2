---
title: Create Required Classes
sidebar_position: 5
description: Create required Java classes for handling push notifications in Android
keywords: [Push Notification Management Tutorial, Push Notification Management, Ant Media Server Documentation, Ant Media Server Tutorials]
---

# Step 5: Create Required Classes

In this step, we will implement the classes needed for handling push notifications, call events, and WebRTC interactions.

## Ant Media Firebase Messaging Service

This service extends `FirebaseMessagingService` and is responsible for handling incoming push notifications and updates to the FCM token.

```java
public class AntMediaFirebaseMessagingService extends FirebaseMessagingService {

        private static final String TAG = "AntMediaFMS";

        public static String fcmToken = "";

        @Override
        public void onMessageReceived(RemoteMessage remoteMessage) {
                Log.d(TAG, "From: " + remoteMessage.getFrom());

                // Check if message contains a notification payload.
                if (remoteMessage.getNotification() != null) {
                        Log.d(TAG, "Message Notification Body: " + remoteMessage.getNotification().getBody());
                }

                // show call notification
                NotificationHelper.showCallNotification(this);
        }

        /**
         * There are two scenarios when onNewToken is called:
         * 1) When a new token is generated on initial app startup
         * 2) Whenever an existing token is changed
         * Under #2, there are three scenarios when the existing token is changed:
         * A) App is restored to a new device
         * B) User uninstalls/reinstalls the app
         * C) User clears app data
         */
        @Override
        public void onNewToken(@NonNull String token) {
                Log.d(TAG, "Refreshed token: " + token);

                fcmToken = token;
        }
}

```

## Accept Call Receiver

Handles the "Accept" button action from the incoming call notification.

```java
public class AcceptCallReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "Call accepted.", Toast.LENGTH_SHORT).show();

        NotificationHelper.dismissCallNotification(context);

        PeerForNotificationActivity.streamId = "streamId";

        Intent callIntent = new Intent(context, PeerForNotificationActivity.class);
        callIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(callIntent);
    }
}
```

## Decline Call Receiver

Handles the "Decline" button action from the incoming call notification.

```java
public class DeclineCallReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "Call declined.", Toast.LENGTH_SHORT).show();

        NotificationHelper.dismissCallNotification(context);

        Intent callIntent = new Intent(context, MainActivity.class);
        callIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(callIntent);
    }
}
```

## Notification Helper

Utility class for managing call notifications with Accept and Decline actions.

```java

public class NotificationHelper {

    private static NotificationManager notificationManager = null;

    private static String callerName = "John Doe";

    private static String roomName = "Room 1";

    public static void setCallerName(String name) {
        callerName = name;
    }

    public static void setRoomName(String name) {
        roomName = name;
    }

    public static void showCallNotification(Context context) {
        notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
        String channelId = "call_notifications";
        String channelName = "Call Notifications";

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel = new NotificationChannel(channelId, channelName, NotificationManager.IMPORTANCE_HIGH);
            notificationManager.createNotificationChannel(channel);
        }

        // Intent for the accept action
        Intent acceptIntent = new Intent(context, AcceptCallReceiver.class);
        PendingIntent acceptPendingIntent = PendingIntent.getBroadcast(context, 0, acceptIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);

        // Intent for the decline action
        Intent declineIntent = new Intent(context, DeclineCallReceiver.class);
        PendingIntent declinePendingIntent = PendingIntent.getBroadcast(context, 0, declineIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);

        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, channelId)
                .setSmallIcon(R.drawable.ic_loopback_call)
                .setContentTitle("Incoming Call")
                .setContentText(callerName+" is calling...")
                .setPriority(NotificationCompat.PRIORITY_HIGH)
                .setCategory(NotificationCompat.CATEGORY_CALL)
                .addAction(R.drawable.ic_loopback_call, "Answer", acceptPendingIntent)
                .addAction(R.drawable.disconnect, "Decline", declinePendingIntent);

        notificationManager.notify(1, builder.build());
    }

    public static void dismissCallNotification(Context context) {
        if (notificationManager != null) {
            notificationManager.cancel(1);
        }
    }
}
```

## Call Notification Activity

Manages push notifications with WebRTC, ensuring compatibility with modern Android versions (API level 33+).

```java
public class CallNotificationActivity extends ComponentActivity {

    String streamId;

    String subscriberId;

    String receiverSubscriberId;

    String authToken;

    String pushNotificationToken;

    String tokenType;

    JSONObject pushNotificationContent;

    JSONArray receiverSubscriberIdArray;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_simple_publish);

        FirebaseApp.initializeApp(this);

        SharedPreferences sharedPreferences =
                PreferenceManager.getDefaultSharedPreferences(this);

        SurfaceViewRenderer fullScreenRenderer = findViewById(R.id.full_screen_renderer);
        String serverUrl = sharedPreferences.getString(getString(R.string.serverAddress), SettingsActivity.DEFAULT_WEBSOCKET_URL);

        IWebRTCClient webRTCClient = IWebRTCClient.builder()
                .setActivity(this)
                .setInitiateBeforeStream(true)
                .setWebRTCListener(createWebRTCListener())
                .setLocalVideoRenderer(fullScreenRenderer)
                .setServerUrl(serverUrl)
                .build();

        streamId = "streamId" + (int)(Math.random()*9999);

        PeerForNotificationActivity.streamId = streamId;

        //Define the subscriberId and it can be any subscriber Id
        subscriberId = "test1@antmedia.io";

        //Define the receiverSubscriberId and it can be any subscriber Id
        receiverSubscriberId = "test2@antmedia.io";

        authToken = "";

        pushNotificationToken = AntMediaFirebaseMessagingService.fcmToken;

        tokenType = "fcm";

        pushNotificationContent = new JSONObject();
        receiverSubscriberIdArray = new JSONArray();

        try {
            pushNotificationContent.put("Caller", subscriberId);
            pushNotificationContent.put("StreamId", streamId);
            receiverSubscriberIdArray.put(receiverSubscriberId);
        } catch (JSONException e) {
            throw new RuntimeException(e);
        }


        askNotificationPermission();
    }

    private IWebRTCListener createWebRTCListener() {
        return new DefaultWebRTCListener() {
            @Override
            public void onWebSocketConnected() {
                super.onWebSocketConnected();
                webRTCClient.registerPushNotificationToken(subscriberId, authToken, pushNotificationToken, tokenType);
                webRTCClient.sendPushNotification(subscriberId, authToken, pushNotificationContent, receiverSubscriberIdArray
                );
            }

        };
    }

    // [START ask_post_notifications]
    // Declare the launcher at the top of your Activity/Fragment:
    private final ActivityResultLauncher<String> requestPermissionLauncher =
            registerForActivityResult(new ActivityResultContracts.RequestPermission(), isGranted -> {
                if (isGranted) {
                    // FCM SDK (and your app) can post notifications.
                } else {
                    // TODO: Inform user that that your app will not show notifications.
                }
            });

    private void askNotificationPermission() {
        // This is only necessary for API level >= 33 (TIRAMISU)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS) ==
                    PackageManager.PERMISSION_GRANTED) {
                // FCM SDK (and your app) can post notifications.
            } else if (shouldShowRequestPermissionRationale(Manifest.permission.POST_NOTIFICATIONS)) {
                // display an educational UI
            } else {
                // Directly ask for the permission
                requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS);
            }
        }
    }
}
```

## Peer For Notification Activity

Handles WebRTC peer-to-peer calls and messaging via data channels.

```java
public class PeerForNotificationActivity extends TestableActivity {

private TextView broadcastingTextView;
public static String streamId;
private IWebRTCClient webRTCClient;

@RequiresApi(api = Build.VERSION_CODES.M)
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_peer_for_notification);

    SurfaceViewRenderer fullScreenRenderer = findViewById(R.id.full_screen_renderer);
    SurfaceViewRenderer pipRenderer = findViewById(R.id.pip_view_renderer);

    broadcastingTextView = findViewById(R.id.broadcasting_text_view);

    String serverUrl = sharedPreferences.getString(getString(R.string.serverAddress), SettingsActivity.DEFAULT_WEBSOCKET_URL);

    webRTCClient = IWebRTCClient.builder()
            .setLocalVideoRenderer(pipRenderer)
            .addRemoteVideoRenderer(fullScreenRenderer)
            .setServerUrl(serverUrl)
            .setActivity(this)
            .setWebRTCListener(createWebRTCListener())
            .setDataChannelObserver(createDatachannelObserver())
            .build();

    Log.i(getClass().getSimpleName(), "Calling play start");
    webRTCClient.join(streamId);
}

private IDataChannelObserver createDatachannelObserver() {
    return new DefaultDataChannelObserver() {
        @Override
        public void textMessageReceived(String messageText) {
            super.textMessageReceived(messageText);
            Toast.makeText(PeerForNotificationActivity.this, "Message received: " + messageText, Toast.LENGTH_SHORT).show();
        }
    };
}

private IWebRTCListener createWebRTCListener() {
    return new DefaultWebRTCListener() {
        @Override
        public void onPlayStarted(String streamId) {
            super.onPlayStarted(streamId);
            broadcastingTextView.setVisibility(View.VISIBLE);
            decrementIdle();
        }

        @Override
        public void onPlayFinished(String streamId) {
            super.onPlayFinished(streamId);
            broadcastingTextView.setVisibility(View.GONE);
            decrementIdle();
        }
    };
}

public void sendTextMessage(String messageToSend) {
    final ByteBuffer buffer = ByteBuffer.wrap(messageToSend.getBytes(StandardCharsets.UTF_8));
    DataChannel.Buffer buf = new DataChannel.Buffer(buffer, false);
    webRTCClient.sendMessageViaDataChannel(streamId, buf);
}
```

## Congratulations!

You've now created all the core classes required to handle push notifications, call events, and WebRTC interactions in your Android app. With these components in place, your app is fully capable of managing incoming calls, interactive notifications, and peer-to-peer communication, giving users a complete real-time communication experience.
