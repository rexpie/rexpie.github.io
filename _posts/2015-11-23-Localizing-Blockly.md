---
comments: true
layout: post
title:  "Localizing Blockly"
tags: Visual Programming
---

# Localizing Blockly within WKwebView

## Add translation for Blockly
Setup the block declaration code using Blockly.Msg.<LOCALIZED_STRING> and prepare
the language files under msg/json folder. When you compile blockly they will get
compiled as well.

## Import translation file in WKwebView
Now import that compiled language file. Of course we don't want to hard-code so
we setup the language in the localizable strings file:

In localizable.strings(Base):

```
  "language" = "en";
```

In localizable.strings(Simplified Chinese):

```
  "language" = "zh-hans";
```

Then we can load the language property into the user script. Note we put this code
at the *End* of the document so it overrides default language source.

```
  let language = NSLocalizedString("language", comment:"")
  if let languageJSPath = NSBundle.mainBundle().pathForResource(language, ofType: "js", inDirectory: "blockly/msg/js") {
    do {
        let languageJS = try NSString(contentsOfFile: languageJSPath, encoding: NSUTF8StringEncoding) as String
        let languageScript = WKUserScript(source: languageJS, injectionTime: .AtDocumentEnd, forMainFrameOnly: true)
        userContentController.addUserScript(languageScript)
    } catch {
        print("Cannot load lang file")
    }
  }
```

## Localizing HTML

Next we should localize HTML elements.  

[This article](http://www.localeplanet.com/support/howto-localize-js.html) is
good practice for JS based dynamic localization. I followed it with a few tweaks.

### Localizing Blockly Toolbox

Blockly Toolbox is defined in the HTML page using XML element. Then `toolbox.js` waltzes
in and dynamically creates a virtual dom. So we should go into the code and intercept
 the creation.

In `toolbox.js`, in function `Blockly.Toolbox.prototype.populate_`, make the
following change:

```
 var childOut = rootOut.createNode(_(childIn.getAttribute('name')));
```

Then use these regex on your HTML to mark the strings:

```
find regex:   category name="([^"]*)"
replace with: category name="_('$1')"
```

Then use gettext to generate the `po` file:

```
xgettext.pl apps/blocklyduino/BigBee.html
```

Use the tools of your choice to translate.

After that, you can generate the messages file using:

```
po2js.py messages.po
```

This creates a messages.js file that you can directly embed in
your HTML:

```
<script type="text/javascript" src="../../translate_compressed.js"></script>
<script type="text/javascript" src="../../messages.js"></script>
```

You may need to install the `polib` for `python`:

```
pip install polib
```

You might wonder,
> But how do we dynamically choose which file should we use for
translation?

Luckly, WKwebView has the `navigation.language` property that we
can use. We can add this snippet to the translation file:

```
var translate = function(text) {
  if (!navigator.language || navigator.language === "en-us") {
    return text;
  }
```


As for the other elements in the HTML file, we can use this
pattern:

```
var buttons = ['btnUpload', 'btnLoadExample', 'btnDiscard'];
buttons.map(function(buttonId){
  document.getElementById(buttonId).innerHTML = _(document.getElementById(buttonId).innerHTML);
});
```

### Optional

You can use
[this link](http://closure-compiler.appspot.com/home) to compile/compress your js code.
