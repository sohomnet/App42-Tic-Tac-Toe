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
13. [Download Face-book SDk] (https://github.com/facebook/facebook-android-sdk) and add it as a library project in your application
14. You can also modify your FB_APP_ID is in Constants.java file.
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

_Get Face-book Friends:__

This has been done in  AsyncApp42ServiceApi.java

```
                  	Social linkObj = socialService.linkUserFacebookAccount(userID,
							accessToken);
		   		Social socialObj = socialService.getFacebookFriendsFromLinkUser(userID);
		       	final ArrayList<Friends> friendList =socialObj.getFriendList();
```

__Register User:__ First register yourself to play game.
 User registration has been done in AsyncApp42ServiceApi.java

```
            		User user = userService.createUser(name, pswd, email);
```
__Authenticate User:__ If you already  registered with App42 than authentication is required .
  User Authentication has been done in AsyncApp42ServiceApi.java

```
           		  App42Response response = userService.authenticate(
							name, pswd);
```
__Push Service registration :__ To get Push notification you have to register your device on APP42 using PushNotificationService.

Device Registration is done in AsyncApp42ServiceApi.java

```
      	  public void registerForPushNotification(Context context,final String userID) {
			GCMRegistrar.checkDevice(context);
			GCMRegistrar.checkManifest(context);
			final String deviceId = GCMRegistrar.getRegistrationId(context);
			if (deviceId.equals("")) {
				GCMRegistrar.register(context, Constants.SenderId);
			} else {
				final Handler callerThreadHandler = new Handler();
				new Thread() {
					@Override
					public void run() {
						try {
							ServiceAPI sp = new ServiceAPI(
									Constants.App42ApiKey,
									Constants.App42ApiSecret);
							String userName = Constants.GameName+ userID;
							pushService.storeDeviceToken(userName, deviceId);
							callerThreadHandler.post(new Runnable() {
								@Override
								public void run() {

								}
							});
						} catch (Exception e) {

						}
					}
				}.start();
			}
	}
```
__Create Game with face-book friends:__ To challenge your friend to play game.
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




__Create Game:__ While starting a new game with opponent you have to create a game.
 Game creation has been done in AsyncApp42ServiceApi.java
 
```
		public void createGameWithFbFriend(final String friendId,final String friendName,final String frindPicUrl,
			final FriendList callBack){

			final Handler callerThreadHandler = new Handler();
			new Thread() {
				@Override
				public void run() {
					try {
						final JSONObject gameObject = new JSONObject();
						gameObject.put(Constants.GameFirstUserKey, UserContext.MyUserName);
						gameObject.put(Constants.GameSecondUserKey, friendId);
					
						gameObject.put(Constants.GameFbName, UserContext.MyDisplayName);
						gameObject.put(Constants.GameFbFriendName, friendName);
					
						gameObject.put(Constants.GameMyPicUrl, UserContext.MyPicUrl);
						gameObject.put(Constants.GameFriendPicUrl, frindPicUrl);
					
						gameObject.put(Constants.GameStateKey,
							Constants.GameStateIdle);
						gameObject.put(Constants.GameBoardKey,Constants.GameIdleState);
						gameObject.put(Constants.GameWinnerKey, "");
						gameObject.put(Constants.GameNextMoveKey, friendId);
		
						gameObject.put(Constants.GameIdKey, java.util.UUID
								.randomUUID().toString());
						storageService.insertJSONDocument(
								Constants.App42DBName,
								Constants.App42UserGamesCollectionPrefix
										+ UserContext.MyUserName, gameObject.toString());
						storageService
								.insertJSONDocument(
										Constants.App42DBName,
										Constants.App42UserGamesCollectionPrefix
												+ friendId,
										gameObject.toString());


						callerThreadHandler.post(new Runnable() {
							@Override
							public void run() {
								callBack.onCreateGame(true,gameObject,friendId);
							}
						});

					} catch (Exception e) {
						callerThreadHandler.post(new Runnable() {
							@Override
							public void run() {
								if (callBack != null) {
									callBack.onCreateGame(false,null,null);
								}
							}
						});
					}
				}
			}.start();
	
		}
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
