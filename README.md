# Xtify-Android-Auto-Example

##Background

This application provides a very simple Android Auto project. It shows how to connect Xtify through to an Android Auto dashboard. This app
can display and read back the notification to the user allowing stores to send out deals and have the user see them as they drive in a safe manner.

##Initial Setup

1. In order to view and build the project you will need to download [Android stuido from Google](https://developer.android.com/sdk/index.html)
2. Next you will need to install the android SDK at API level 21.
3. You will also need to download the Android support library and repository.
4. The remainder of the dependencies should be included and specified in the Gradle file. They are as follows:
  * android-support-multidex.jar
  * xtify-android-sdk-2.6.4.jar
  
##App Configuration

In order to enable both android Auto and Xtify in the application certain modifications must be made to the Android Manifest file.

###Permissions

The following permissions are required for this application:

```
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <permission android:name="com.ibm.mil.milandroidautoxtifyexample.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="com.ibm.mil.milandroidautoxtifyexample.permission.C2D_MESSAGE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```
Where `com.ibm.mil.milandroidautoxtifyexample` will be your specific app package id.

###Android Auto

To have Andorid Auto enabled you will need to specify the following components:

```         
    <meta-data
    android:name="com.google.android.gms.car.application"
    android:resource="@xml/automotive_app_desc" />
```
This specifies the file that says what features of Android Auto will be used in this application. The contents of this file are:

```
    <automotiveApp xmlns:android="http://schemas.android.com/apk/res/android">
        <uses name="notification" />
    </automotiveApp>
```
Which states that this application is using the notificaiton features of Andorid Auto. You will also need to specify Message Heard and Reply recievers if you want the app to track
when a user listens to a message and if you want the user to be able to reply to it:

```
    <receiver
        android:name=".MyMessageHeardReceiver"
        android:enabled="true"
        android:exported="true" >
    </receiver>
    <receiver
        android:name=".MyMesageReplyReceiver"
        android:enabled="true"
        android:exported="true" >
    </receiver>
```

The actual sending of the notification to the Android Auto dashboard is done through the `XtifyNotifier` class. It consists of constructing Pending intents, building a conversation
and sending it over to the dashboard. For more information on devleoping for Android Auto view [Google's Official Documentation](https://developer.android.com/training/auto/start/index.html)

###Xtify

To enable Xtify you need to add the following providers, recievers and services to the `AndroidManifest.xml`:

```
    <provider
       android:name="com.xtify.sdk.db.Provider"
       android:authorities="com.ibm.mil.milandroidautoxtifyexample.XTIFY_PROVIDE
	   android:exported="false" />
       <receiver android:name="com.xtify.sdk.c2dm.C2DMBroadcastReceiver" >
            <intent-filter android:permission="com.google.android.c2dm.permission.SEND" >
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="mil.ibm.com.loyaltyauto" />
            </intent-filter>
            <intent-filter android:permission="com.google.android.c2dm.permission.SEND" >
                <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
                <category android:name="mil.ibm.com.loyaltyauto" />
            </intent-filter>
        </receiver>
        <receiver android:name="com.ibm.mil.milandroidautoxtifyexample.XtifyNotifier" >
            <intent-filter>
                <action android:name="com.xtify.sdk.NOTIFIER" />
            </intent-filter>
        </receiver>
        <receiver android:name="com.xtify.sdk.NotifActionReceiver" />
        <receiver android:name="com.xtify.sdk.wi.AlarmReceiver" >
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.TIMEZONE_CHANGED" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.PACKAGE_REPLACED" />
                <data android:scheme="package" />
            </intent-filter>
        </receiver>

        <service android:name="com.xtify.sdk.location.LocationUpdateService" />
        <service android:name="com.xtify.sdk.c2dm.C2DMIntentService" />
        <service android:name="com.xtify.sdk.alarm.MetricsIntentService" />
        <service android:name="com.xtify.sdk.alarm.TagIntentService" />
        <service android:name="com.xtify.sdk.alarm.RegistrationIntentService" />
        <service android:name="com.xtify.sdk.alarm.LocationIntentService" />
```
This enables both specific fetures needed in the Xtify SDK and Google notifications in general. The line `<receiver android:name="mil.ibm.com.loyaltyauto.XtifyNotifier" >`
specifies the class that will respond to notification and registration events. To regiter the device with Xtify just override the onStart function in the activity you want
registration to occur like so:

```
@Override
    protected void onStart() {
        super.onStart();
        XtifySDK.start(getApplicationContext(), XTIFY_APP_KEY, PROJECT_NUM);
    }
```



