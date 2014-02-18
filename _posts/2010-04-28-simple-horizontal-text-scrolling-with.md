---
layout: post
title: "Simple horizontal text scrolling with Ext Core 3 (marquee replacement)"
description: ""
category: Devel
tags: [javascript, ExtJS]
---
For one of my projects I needed to implement a line of scrolling text at the top or at the bottom of the browser window. One of the options was to use the non-standard [marquee HTML element](http://en.wikipedia.org/wiki/Marquee_element), which I rejected for obvious reasons. Plus, it didn't work very well - the motion was not smooth and there were too few options to customize its behaviour. So I decided to implement the text scrolling using Javascript.

 There are plenty of code examples on the web. In fact, it's fairly easy, you just need to update the text position using CSS properties. Since I'm used to the [ExtJS framework](http://www.sencha.com/products/extjs/) i wrote a simple class using the [Ext Core 3](http://www.sencha.com/products/extcore/) library. Actually I didn't need the high-level features of ExtJS, but I wanted to take advantage of the cross-borswer support, that's why I used the low-level library Ext Core.


Here it is. You can use it as an inspiration. View a simple [demo here](http://www.debug.cz/examples/js/marquee/).

```js
/**
 * Simple class for horizontal scrolling of text in a box 
 * written for Ext Core 3
 * 
 * @author Ivan Novakov 
 * @link http://www.debug.cz/examples/js/marquee/
 */

Ext.namespace('Ext.ux');

/**
 * @class Ext.ux.Marquee
 * @param {} config
 */
Ext.ux.Marquee = function(config) {
    Ext.apply(this, {
        // the scroll step in pixels
        step : 5,
        // the update interval
        interval : 100,
        // the CSS class of the container
        containerCls : 'undefined-container-class',
        // the CSS class of the text element
        textCls : 'undefined-text-class',
        // the element ID of the text element
        textElmId : 'mtext',
        text : [
            'undefined text'
        ]
    });

    Ext.apply(this, config);

    this.currentTextIndex = -1;
    this.textElm = null;
    this.currentTask = null;
    this.taskRunner = null;

    this.addEvents({
        beforetextupdate : true
    });
};

/**
 * @class Ext.ux.Marquee
 * @extends Ext.util.Observable
 */
Ext.extend(Ext.ux.Marquee, Ext.util.Observable, {

    _initDom : function() {
        var elm = Ext.DomHelper.append(document.body, {
            tag : 'div',
            cls : this.containerCls,
            children : [
                {
                    tag : 'span',
                    id : this.textElmId,
                    cls : this.textCls,
                    html : ' '
                }
            ]
        });
    },

    init : function() {
        this._initDom();
        this.textElm = this._getTextElm();
        var text = this._getCurrentText();

        this._resetTextElm();

        this.currentTask = {
            run : this.move,
            interval : this.interval,
            scope : this
        };

        this.taskRunner = new Ext.util.TaskRunner();
        this.taskRunner.start(this.currentTask);
    },

    move : function() {
        if (this.textElm.getRight() <= 0) {
            this.fireEvent('beforetextupdate');
            this._resetTextElm();
        }

        var left = this.textElm.getX() - this.step;
        this.textElm.setX(left);
    },

    _resetTextElm : function() {
        this.textElm.setX(this.textElm.parent().getRight());
        this.textElm.update(this._getNextText());
    },

    _getNextText : function() {
        this._incrementTextIndex();
        return this._getCurrentText();
    },

    _getCurrentText : function() {
        return this.text[this.currentTextIndex];
    },

    _incrementTextIndex : function() {
        if (this.currentTextIndex >= this.text.length - 1) {
            this.currentTextIndex = 0;
        } else {
            this.currentTextIndex++;
        }
    },

    _getTextElm : function() {
        return Ext.get(this.textElmId);
    }

});
```

To use it we have to define these CSS:

```css
.marquee-container {
  position: fixed;
  bottom: 0px;
  left: 0px;
  right: 0px;
  background-color: #ddd;
  padding: 0px;
  width: 100%;
  height: 60px;
  background-color: #333;
  text-align: left;
}

.marquee-text {
  position: absolute;
  vertical-align: baseline;
  font-size: 40px;
  font-weight: bold;
  color: #ddd;
  white-space: nowrap;
  margin: 5px;
  font-family: Trebuchet MS, Lucida Sans Unicode, Arial, sans-serif;
}
```

And to add this javascript code:

```js
Ext.onReady(function() {

    var marquee = new Ext.ux.Marquee({
        step: 4,
        interval: 30,
        containerCls: 'marquee-container',
        textCls: 'marquee-text',
        text: [
            'This is a simple example of the scrolling',
            'If you have any questions, write me - ivan.novakov [at] debug [dot] cz',
            'Enjoy!'
        ]
    });
    
    marquee.init();
});
```