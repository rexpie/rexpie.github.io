---
comments: true
layout: post
title:  "Hacking Blockly"
tags: Visual Programming, Blockly
---

# Blockly Touch/Drag Events Tweaking


## The issue

> TL,DR: I want to enable scrolling with dragging on the flyout menu.


I recently played with [BlocklyDuino](https://github.com/BlocklyDuino/BlocklyDuino) a bit, and it was fun. I even embedded it into IOS app, using WKWebview and some JS bridging delegate.  
Only problem is when I added too many blocks into one toolbox category, I need to scroll down the flyout menu to find what I want. Now on touch devices, scrolling is fairly popular interaction and users have high expectations for that. But the blockly flyout is not doing what I expect. As soon as you touch one block, a new block is immediately created and the flyout menu closes. You can only scroll by dragging the scrollbar handle, or you touch empty spaces between or beside the blocks and drag the scroll panel. These spaces are very tight for smaller devices, making it hard to manipulate the menu.

## The culprit

After some code searching, the flyout.js file shows that the menu from toolbox has an `autoClose` attribute. So first,

```javascript
Blockly.Flyout.prototype.autoClose = false;
```

Then, we go to the place where new blocks are created, and change its behaviour:

```javascript
// only create a new block when dragging towards the right. Try to scroll up and down when dragging vertically.
if (Math.sqrt(dx * dx + dy * dy) > Blockly.DRAG_RADIUS && dx > 2 * Math.abs(dy)) {
  // Create the block.
  Blockly.Flyout.startFlyout_.createBlockFunc_(Blockly.Flyout.startBlock_)(Blockly.Flyout.startDownEvent_);
} else {
  // dragging menu vertically
  var metrics = Blockly.Flyout.startFlyout_.getMetrics_();
  var y = metrics.viewTop - dy;
  y = Math.min(y, metrics.contentHeight - metrics.viewHeight);
  y = Math.max(y, 0);
  Blockly.Flyout.startDownEvent_ = e;
  Blockly.Flyout.startFlyout_.scrollbar_.set(y);
}
```

Also, we need to change the code where the flyout closes itself, this code is near the end of function `createBlockFunc_`. The tricky part is that the flyout menu for the context menu is also using this bit of code.

```javascript
// Check if the flyout menu is popped out by toolbox or not. Do not hide the menu if context menu is calling this flyout.
if (workspace.toolbox_ && workspace.toolbox_.tree_) {
  flyout.hide();
  workspace.toolbox_.clearSelection();
}
```

And we are all set. The menu is scrollable via dragging vertically on the blocks as well. When you drag a block to the right, a new block is created and the flyout menu also closes itself.
