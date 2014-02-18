---
layout: post
title: "Using the window.onerror Handler For Logging Javascript Errors"
description: ""
category: Devel
tags: [javascript]
---
I'm running a really large administration application with javascript based user interface. It is used by a large number of users on different computers with different operation systems and different browsers. Sometimes weird javascript errors occur. And when such error occurs, the browser stops the execution of the script and the users usually reports something like "it didn't work".

Luckily, there is a way how to handle this and although it may not cover 100% of all the errors, it might be helpful when dealing with errors at the client side. If you define the `window.onerror` handler, it will be executed when an error occurs. Three arguments are passed to the handler - the error message, the URL of the script and the line, where the error is located.

Now, the question is, what to do. The ship is sinking and you are allowed to perform one last action. It depends on the application, but generally you could send info about the error to the server, where it can be logged. You could also notify the user, that an error occurred and the application probably won't work as expected and advise him, what to do next.

Here is a simple example of how the `window.onerror` handler can be used to send the error information to the server:

```js
/*
 * window.onerror handler
 */
window.onerror = function(message, url, line) {
    if (window.XMLHttpRequest) {
        var xhr = new XMLHttpRequest();
        
        // the url to post data to
        var logurl = "ajax/system/jsError";

        // serialize the POST params
        var params = 'message=' + message + '&url=' + url + '&line=' + line + '&userAgent='
        + window.navigator.userAgent;

        // open an asynchronous connection
        xhr.open("POST", logurl, true);

        // set the appropriate headers
        xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
        xhr.setRequestHeader("Content-Length", params.length);
        xhr.setRequestHeader("Connection", "close");

        // send the request
        xhr.send(params);
    }

    // returning false triggers the execution of the built-in error handler
    return false;
}
```

Make sure, that the code is executed before anything else. Even if you use a javascript framework, I would recommend not to use its code in the handler body. It is possible, that the error occurs before all the components has been loaded and you'll got an error in your error handler :). At least it seems that in most browsers errors in the error handler don't trigger the error handler, so loops won't occur.

One more notice - if there is a parse error in the script, where you defined the error handler, it won't be triggered. Parse errors in other scripts or runtime errors in the same script are OK :).
