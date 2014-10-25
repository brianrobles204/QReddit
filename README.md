#QReddit

QML Library for Reddit.

QReddit uses javascript with QML objects to create a simple way to connect to Reddit from your QML app. It is designed to be event driven, using Qt signals to notify your app when the connection has succeeded (or failed). QReddit supports multiple users, downloading and voting on posts and comments. However, the library is still incomplete and lacks user profile and messaging, among other things.

--------------

###Installation

Include the folder in your project directory and add an import statement to your QML files

-------------

###Usage

Create a new QReddit object:

```
import "QReddit/QReddit.js" as QReddit

property var redditObj: new QReddit.QReddit("User Agent", "application-name")
```

####ConnectionObject

The core of QReddit the the ConnectionObject. Almost any function that requires connecting to Reddit returns this QML object. It emits a `success` signal when the request is successful and Reddit reports no input errors. Hook into this signal when updating the app display, and use the `response` property to get the output parsed by QML. ConnectionObject also emits an `error` signal, and a `raiseRetry` signal when the server is taking too long to respond. You may call the `abort` signal to abort the connection. See `ConnectionObject.qml` for more documentation.

####Users

QReddit supports storing and switching between multiple users, but only allows one user to be logged in at a time.

To login a new user,
```javascript
//The loginNewUser function returns a connection object.
var loginConnObj = redditObj.loginNewUser("username", "password")
loginConnObj.onSuccess.connect(function(){
    redditObj.updateSubscribedArray() //Populates the user's list of subscribed subreddits
    /* Refresh the home page display */
})
loginConnObj.onError.connect(function(errorMessage){
    /* Show the login error to the user. 
       Perhaps the password is wrong, or there's no internet. */
})
```
The new user will be set as the active user for the session. Any calls that require a logged in user can now be completed. The session ends when your app is closed. Later, if you need to log in the active user again, simply use
```javascript
redditObj.loginActiveUser()
```

If you have multiple users saved, you can get a list of their usernames
```javascript
var usernameArray = redditObj.getUsers()
var activeUsername = redditObj.getActiveUser()
```

and you can switch between them
```javascript
var switchConnObj = redditObj.switchActiveUser("newusername")
```
This function will only be successful if the given username has already been stored (i.e. logged in before).

You can also access the user's subreddits
```javascript
//Returns a connection object containing the active user's subscribed subreddits
var subsConnObj = redditObj.updateSubscribedArray()

//Returns the stored array of strings containing the user's subscribed subreddits
var subsArray = redditObj.getSubsArray()
```
You can also pass the usernames of any stored user to access to the aforementioned functions to access their subreddits too.

See `QReddit.js` for a complete list of user-related functions

####Session

QReddit has a NotifierObject that emits updates relating to the Reddit session. Changes in the active user, and updates in the active user's subscribed subreddits will be notified here. For example:
```
property var redditNotifier: redditObj.notifier

...

ActivityIndicator {
    ...
    running: redditNotifier.authStatus === "loading"
    ...
}

...

ToolbarButton {
    action: Action {
        ...
        enabled: redditNotifier.isLoggedIn
        ...
    }
}
```
