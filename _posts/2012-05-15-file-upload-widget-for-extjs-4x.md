---
layout: post
title: "File Upload Widget For ExtJS 4.x"
description: ""
category: Devel
tags: [javascript, ExtJS]
---
In a project I'm working on, I needed a flexible file upload panel with multiple file upload support. Unfortunately, the ExtJS built-in mechanisms ([Ext.form.field.File](http://docs.sencha.com/extjs/4.1.3/#!/api/Ext.form.field.File)) does not support multiple upload and there are some other limitations as well. I searched for a user contributed plugin, but I couldn't find a suitable one. Most of the contributions were written for older versions of ExtJS - 3.x or even 2.x. So I decided to write one myself, trying to use some advanced features of HTML5.

You may first check the [live demo](http://debug.cz/demo/upload/).

I had to focus on two main aspects:

* user interface - the way it looks like and interacts with the user (dialogs, grids, buttons, status bars, ...)
* data transport - the upload itself, how to transport data to the server

In my case, the first aspect was very much influenced by the fact, that I needed the widget to fit the whole application. So perhaps the layout won't suit everyone. I'm planing to improve the flexibility of the user interface in the future.

Regarding the second aspect - the transport method - I wanted to separate the implementation in a way, that will allow easy deployment of alternative methods in the future. For the time being I implemented just one method - raw PUT/POST with the following features:

* uses the standard ExtJS mechanisms - `Ext.data.Connection` and its `XMLHttpRequest`
* doesn't require hidden iframes, flash objects or other workarounds - it's just plain AJAX
* sends data in the body of the request and metadata (filename, size, type) in the headers
* provides upload progress data
* allows uploading of really large files (tested with a 2GB file - works best in Chrome, Mozilla Firefox had problems, because it apparently tries to read the whole file into the memory first)

You may try the initial version available on github. For now, it's not as flexible as I would like it to be, but perhaps it will be useful for someone. To be able to use it in your application you just need to clone the repository somewhere on your system and add the path with the prefix to `Ext.Loader`:

```js
Ext.Loader.setPath({
    'Ext.ux.upload' : '/my/path/to/extjs-upload-widget/lib/upload'
});
And then, you can use the dialog:
var dialog = Ext.create('Ext.ux.upload.Dialog', {
    dialogTitle: 'My Upload Widget',
    uploadUrl: 'upload.php'
});

dialog.show();
```

For more options you can check the [API docs](http://debug.cz/demo/upload/docs/#!/api/Ext.ux.upload.Dialog).

Links:

* [the github project](https://github.com/ivan-novakov/extjs-upload-widget)
* [live demo (included in the repository)](http://debug.cz/demo/upload/)
* [API docs](http://debug.cz/demo/upload/docs/#!/api/Ext.ux.upload.Dialog)