---
comments: true
layout: post
title:  "Blockly Multitouch Bug Fix"
tags: Visual Programming
---

# The Bug

I am building a mobile visual programming app for [Bluno](http://www.dfrobot.com/index.php?route=product/product&product_id=1044#.UoyIUpH7k8M) and its compatible devices.
Long story short, there is a bug on both IOS and Android that convinces me that Blockly is not doing things right.

When I swipe my fingers across Blockly workspace, sometimes the rest of the HTML page stops responding to taps or swipes. At first I thought it was issue of event binding of `touchStart` and `mouseDown`. I changed all my `onClick` attributes to `Blockly.bindEvent_` calls. It worked for some time until I can't scroll the Arduino code text view.

I spent two days debugging and found the bug.


## The fix
On a touch device with multitouch support, multiple `touchStart` events can be fired together. This causes multiple calls to `Blockly.WorkspaceSvg.prototype.onMouseDown_` event handler.

```
    if ('mouseup' in Blockly.bindEvent_.TOUCH_MAP) {
      Blockly.onTouchUpWrapper_ =
          Blockly.bindEvent_(document, 'mouseup', null, Blockly.onMouseUp_);
    }
    Blockly.onMouseMoveWrapper_ =
        Blockly.bindEvent_(document, 'mousemove', null, Blockly.onMouseMove_);
```

This code binds multiple events to the document element but only the last call actually registers the event handler wrapper, which will consequently be called in the global `Blockly.onMouseUp_` handler.

This can cause event handlers for `mouseMove` and `mouseDown` linger in the `document` node, causing other elements on the DOM tree to disfunction.

The fix is relatively straightforward.
