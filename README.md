# Bluemix Mobile Services - Push Notifications Server-side SDK for Java
[![Build Status](https://travis-ci.org/ibm-bluemix-mobile-services/bms-pushnotifications-serversdk-java.svg?branch=master)](https://travis-ci.org/ibm-bluemix-mobile-services/bms-pushnotifications-serversdk-java)
[![Build Status](https://travis-ci.org/ibm-bluemix-mobile-services/bms-pushnotifications-serversdk-java.svg?branch=development)](https://travis-ci.org/ibm-bluemix-mobile-services/bms-pushnotifications-serversdk-java)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/fe43788a157c4c4b971a8918d29c4469)](https://www.codacy.com/app/ibm-bluemix-mobile-services/bms-pushnotifications-serversdk-java?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=ibm-bluemix-mobile-services/bms-pushnotifications-serversdk-java&amp;utm_campaign=Badge_Grade)

The Push Notifications Server-side SDK for Java is used to send push notifications to registered devices through the [Push Notifications service in IBM® Bluemix](https://console.ng.bluemix.net/docs/services/mobilepush/index.html).

## Getting the SDK

You can get the SDK from Maven Central. For example, to get it with Maven, include the following in your dependencies:

```
<dependency>
	<groupId>com.ibm.mobilefirstplatform.serversdk.java</groupId>
	<artifactId>push</artifactId>
	<version>1.0.1</version>
</dependency>
```

## Sending a push notification

To send a push notification, first initialize the SDK with the Bluemix region of your application, and optionally, your credentials:

```
PushNotifications.init("YOUR_APPLICATION_ID", "YOUR_SECRET", PushNotifications.US_SOUTH_REGION);
```

However, if your server is running on Bluemix as well, you can initialize it with just the region. 

You can achieve this by binding your Push Notification service to your server application in Bluemix, which will then give it access to the service's credentials. To do this, just initialize like this:

```
PushNotifications.init(PushNotifications.US_SOUTH_REGION);
```

Then create a new notification using `NotificationBuilder`:

```
NotificationBuilder builder = new NotificationBuilder("This is the notification's text!");
```
You can also configure the notification with some other optional settings, such as:

We use SettingBuilder for creating all optional settings, BuilderFactory to get builder (FirefoxWebBuilder, ApnsBuilder, GcmBuilder etc) and TargetBuilder for constructing Target..

Functionality added for FirefoxWeb, ChromeWeb, SafariWeb, ChromeAppExtension and extral optional settings introduced for Apns and GCM
```


builder.setMessageURL(urlToBeIncludedWithThePushNotification)
	.setTarget(deviceIdArray, userIdArray, platformArray, tagNameArray)
	.setAPNSSettings(badge, category, iosActionKey, payload, soundFile, APNSNotificationType)
	.setGCMSettings(collapseKey, delayWhileIdle, jsonPayload, priority, soundFile, secondsToLive);
	.setTarget(new String [] {"deviceId1","deviceId1"}, new String [] {"user1","user2"}, new PushNotificationsPlatform[] { PushNotificationsPlatform.APPEXTCHROME,
				PushNotificationsPlatform.APPLE, PushNotificationsPlatform.GOOGLE}, new String[] {"tag1","tag2"}); // This approach is deprecated

// Below approach has been introduced

// Setting up all the builders (apns, gcm , firefox etc) in the SettingsBuilder of Settings

Settings settings = new Settings.SettingsBuilder().setApnsBuilder(BuilderFactory.getBuilder(ApnsBuilder.class))
				.setGcmBuilder(BuilderFactory.getBuilder(GcmBuilder.class)).setChromeAppExtBuilder(BuilderFactory.getBuilder(ChromeAppExtBuilder.class))
				.setChromeWebBuilder(BuilderFactory.getBuilder(ChromeWebBuilder.class)).setFirefoxWebBuilder(BuilderFactory.getBuilder(FirefoxWebBuilder.class))
				.setSafariWebBuilder(BuilderFactory.getBuilder(SafariWebBuilder.class)).build();

	// Set all optional settings using settings, you no need to set all attribute of a setting , only which is required you can set using builder for instance for apns if you require only bade as an optional setting , other settings not need to be set.
	
	// For APns Settings
	builder.setAPNSSettings(settings.getApnsBuilder().setBadge(1).setInteractiveCategory("testInteractiveCategory")
				.setIosActionKey("testiOSActionKey").setPayload(new JSONObject()).setSound("testSoundFile")
				.setType(APNSNotificationType.DEFAULT).setTitleLocKey("testTitleLocKey").setLocKey("testLocKey")
				.setLaunchImage("testLaunchImage")
				.setTitleLocArgs(new String[] { "testTitleLocArgs1", "testTitleLocArgs2" })
				.setLocArgs(new String[] { "testLocArgs1", "testLocArgs" }).setTitle("testTitle")
				.setSubtitle("testSubtitle").setAttachmentUrl("testAttachmentUrl").build());


		// style and lights attibute addded to Gcm optional settings which can be constructed as shown below:
				
		Gcm gcm = settings.getGcmBuilder().setGcmStyleBuilder(BuilderFactory.getBuilder(GcmStyleBuilder.class)).setGcmLightsBuilder(BuilderFactory.getBuilder(GcmLightsBuilder.class)).build();
		
		
		GcmStyle style = gcm.getGcmStyleBuilder().setType(GcmStyleTypes.BIGTEST_NOTIFICATION).build();		
				
		
		GcmLights lights = gcm.getGcmLightsBuilder().setLedArgb(GcmLED.BLACK).build();
		
		// Gcm Settings	
		builder.setGCMSettings(settings.getGcmBuilder().setCollapseKey("testCollapseKey").setDelayWhileIdle(true)
				.setPayload(new JSONObject()).setPriority(GCMPriority.MIN).setSound("testSoundFile").setTimeToLive(42)
				.setIcon("testIcon").setVisibility(Visibility.PUBLIC).setSync(true)
				.setStyle(NotificationBuilder.generateJSON(style))
				.setLights(NotificationBuilder.generateJSON(lights)).build());

		// Chrome Settings		
		builder.setChromeSettings(settings.getChromeWebBuilder().setTitle("testTitle").setIconUrl("testIconUrl")
				.setTimeToLive(42).setPayload(new JSONObject()).build());

		// ChromeAppExtension settings		
		builder.setChromeAppExtSettings(settings.getChromeAppExtBuilder().setCollapseKey("testCollapseKey")
				.setDelayWhileIdle(true).setTitle("testTitle").setIconUrl("testIconUrl").setTimeToLive(42)
				.setPayload(new JSONObject()).build());

		// Firefox Settings		
		builder.setFirefoxWebSettings(settings.getFirefoxWebBuilder().setTitle("testTitle").setIconUrl("testIconUrl")
				.setTimeToLive(42).setPayload(new JSONObject()).build());

		// Safari Settings		
		builder.setSafariWebSettings(settings.getSafariWebBuilder().setTitle("testTitle")
				.setUrlArgs(new String[] { "testUrlArgs1", "testUrlArgs2" }).setAction("testAction").build());			




	
```
(Note that you can chain the different calls to `NotificationBuilder`.)

Finally, pass this notification using `NotificationBuilder.build()` and an optional `ResponseListener` to get notified of the result:

```
	PushNotifications.send(notification, new PushNotificationsResponseListener(){

		@Override
		public void onSuccess(int statusCode, String responseBody) {
			System.out.println("Successfully sent push notification! Status code: " + statusCode + " Response body: " + responseBody);
			
		}

		@Override
		public void onFailure(Integer statusCode, String responseBody, Throwable t) {
			System.out.println("Failed sent push notification. Status code: " + statusCode + " Response body: " + responseBody);
				
			if(t != null){
				t.printStackTrace();
			}
		}
			
	});
```

If you have a device that is registered to receive push notifications, you will now receive the notification that was sent.

## License

Copyright 2016 IBM Corp.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
