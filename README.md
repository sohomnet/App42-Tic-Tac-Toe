App42-Tic-Tac-Toe
===========================

# About Game

1. It’s a multi-player turn based game.
2. The winning motive of the game is to connect three respective cross or circle in a vertical, horizontal or diagonal direction.
3. When opponent played his turn, a notification message comes to player that now his turn came.

# Running Sample

This is a sample Android gaming app is made by using App42 backend platform. It uses user, storage, push Notification and gaming APIs of App42 platform. 
Here are the few easy steps to run this sample app.


1. [Register] (https://apphq.shephertz.com/register) with App42 platform
2. Create an app once you are on Quick start page after registration.
3. Go to dashboard and create a new game Tic-tac-toe.
4. If you are already registered, login to [AppHQ] (http://apphq.shephertz.com) console and create an app from App Manager Tab and do step #3 to create a game.
5. [To use Push Notification see video] (http://www.youtube.com/watch?feature=player_embedded&v=4FtpoRkPuPo).
6. To use push Notification service in your application go to https://code.google.com/apis/console,create a new project here.
7. Click on services option in Google console and enable Google Cloud Messaging for Android service.
8. Click on API Access tab and create a new server key for your application with blank server information.
9. Go to AppHQ console and click on Push Notification and select Android setting in Settings option.
10. Select your game and copy server key that is generated by using Google apt console, and submit it.
11. Download the eclipse project from this repo and import it in the same.
12. Open Constants.java in sample app and give the value of app42APIkey app42SecretKey that you have received in step 2 or 4
13. Build and Run 



# Design Details:

__Initialize Services:__

Initialization has been done in AsyncApp42ServiceApi.java

```
        		ServiceAPI sp = new ServiceAPI(Constants.App42ApiKey,
  				Constants.App42ApiSecret);
				this.userService = sp.buildUserService();
				this.storageService = sp.buildStorageService();
				this.pushService = sp.buildPushNotificationService();
```

__Register User:__ First register yourself to play game.
 User registration has been done in in AsyncApp42ServiceApi.java

```
            		 User user = userService.createUser(name, pswd, email);
```
__Authenticate User:__ If you already  registered with App42 than authentication is required .
 User Authentication  has been done in AsyncApp42ServiceApi.java

```
           		  App42Response response = userService.authenticate(
							name, pswd);
```
__Push Service registration :__ To get Push notification you have to register your device on APP42 using PushNotificationService.

Device Registration is done in MainActivty.java

```
      	   public void doRegistration(Context context, final String userID) {
				this.context = context;
				GCMRegistrar.checkDevice(context);
				GCMRegistrar.checkManifest(context);
				final String deviceId = GCMRegistrar.getRegistrationId(context);
				if (deviceId.equals("")) {
				//Sender Id is equals to our project No generated on Google Api console. 
					GCMRegistrar.register(MainActivity.this, Constants.SENDER_ID);
				} else {
						mRegisterTask = new AsyncTask<Void, Void, Void>() {
						@Override
						protected Void doInBackground(Void... params) {
							try {
								ServiceAPI sp = new ServiceAPI(
										Constants.App42ApiKey,
										Constants.App42ApiSecret);
								String userName = Constants.GameName + userID;
								PushNotificationService push = sp
										.buildPushNotificationService();
								push.storeDeviceToken(userName, deviceId);
							} catch (Exception e) {
							}
							return null;
						}

						@Override
						protected void onPostExecute(Void result) {
							mRegisterTask = null;

						}

					};
					mRegisterTask.execute(null, null, null);
				}
			}
```


__Create Game:__ While starting a new game with opponent you have to create a game.
 Game creation has been done in AsyncApp42ServiceApi.java
```
                    final JSONObject gameObject = new JSONObject();
					gameObject.put(Constants.GameFirstUserKey, uname1);
					gameObject.put(Constants.GameSecondUserKey, remoteUserName);
					gameObject.put(Constants.GameStateKey,
							Constants.GameStateIdle);
					gameObject.put(Constants.GameBoardKey,
							Constants.GameIdleState);
					gameObject.put(Constants.GameWinnerKey, "");
					gameObject.put(Constants.GameNextMoveKey, uname1);
					gameObject.put(Constants.GameIdKey, java.util.UUID
							.randomUUID().toString());

					// Insert in to user1's game collection
					storageService.insertJSONDocument(Constants.App42DBName,
							Constants.App42UserGamesCollectionPrefix + uname1,
							gameObject.toString());
					// Insert in to user2's game collection
					storageService.insertJSONDocument(Constants.App42DBName,
							Constants.App42UserGamesCollectionPrefix
									+ remoteUserName, gameObject.toString());
```

__Update Game:__ While playing game.
  Game updating has been done in AsyncApp42ServiceApi.java
```
                    final JSONObject gameObject = new JSONObject();
					gameObject.put(Constants.GameFirstUserKey, uname1);
					gameObject.put(Constants.GameSecondUserKey, remoteUserName);
					gameObject.put(Constants.GameStateKey,
							Constants.GameStateIdle);
					gameObject.put(Constants.GameBoardKey,
							Constants.GameIdleState);
					gameObject.put(Constants.GameWinnerKey, "");
					gameObject.put(Constants.GameNextMoveKey, uname1);
					gameObject.put(Constants.GameIdKey, java.util.UUID
							.randomUUID().toString());

					// Insert in to user1's game collection
					storageService.insertJSONDocument(Constants.App42DBName,
							Constants.App42UserGamesCollectionPrefix + uname1,
							gameObject.toString());
					// Insert in to user2's game collection
					storageService.insertJSONDocument(Constants.App42DBName,
							Constants.App42UserGamesCollectionPrefix
									+ remoteUserName, gameObject.toString());
```

__Push message:__ Once you have played your turn , you have to send notification to your opponent
 Push message has been sent in AsyncApp42ServiceApi.java

```
            		pushService.sendPushMessageToUser(Constants.GameName
							+ userName, newGameObj.toString());
```
