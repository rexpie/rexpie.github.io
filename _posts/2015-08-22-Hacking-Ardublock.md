---
comments: true
layout: post
title:  "Hacking Ardublock"
tags: Visual Programming
---

## Adding an extension to Ardublock

Extended From: [How to hack ardublock](http://blog.ardublock.com/2012/05/04/how-to-hack-ardublock/)

### Handling the dependencies

1. Fork and clone Arduino form github
2. Import openblocks and Ardublock into eclipse
3. Import arduino-builder,arduino-dore and processing from Arduino project into eclipse
4. Modify the buildpath of Ardublock and add the above 3 projects as dependency. Add openblocks and an dependency as well.
5. Modify pom.xml in Ardublock，delete openblocks and Arduino dependency entries(They became obsolete over time and was no longer maintained)
6. Continue to delete the install-arduino-pde execution entry in pom.xml

You should now see no compiling errors within the projects. You may have a few warnings about the java compiler version. Just change the compilation configs to make eclipse happy.

**If you are using other IDEs like IntelliJ, please use your own judgement to appropriately adapt the steps.

### Adding an LED module


You can use the Ardublock article to do the same, here are some basic ideas:

1. Add one BlockGenus with your own parameters
2. Register the BlockGenus into one BlockDrawer so that you'll see the newly added module in the GUI, inside the designated drawer menu.
3. If you need new image, tag or other custom resources to go with the new module, you need to add them to the resources folder, with i18n entries. Modify the `ardublock<_{country_code}>.properties` i18n file to achieve that.
4. Modify block-mapping.properties and point the module to use the corresponding `Translator` Java class. You need to extend the `Translator` class if you want to dive in deeper.

### Export and install

1. Right click Ardublock and Openblocks projects and export a jar package
2. You need to check the `export with all resources` checkbox to include the image and configuration files.
3. Put your jar into the Arduino tools folder. Be careful with the folder names as one of them is said to be case sensitive. See the [official tutorial](http://blog.ardublock.com/2012/05/04/how-to-hack-ardublock/) for sure.


### Test

1. Open the Arduino installation folder
2. Run Arduino-debug and you can see the logs in the console window.
3. Click on `Tools`->`Ardublock`
4. Drag your new module and test the functions.


## Adding an I2C module


***You need to know how to add your custom header file and c++ file***

If you want to use I2C modules like an I2C mounted 1602 LCD, you need to add the I2C c++ libraries for easier coding. The original Ardublock can only generate a single ino file. Here is how:

1. First you create your new module, of course. See steps above. Let's call the new module I2C.
2. Modify the `Translator` base class and add a `customHeaderFileSet` private member, maybe next to its own `headerFileSet` member.
3. Add a `addCustomHeaderFile` API for clients and derived classes to manipulate the `customHeaderFileSet` 
4. Modify your I2C module and call your API to add the custom header and/or c++ file.

{% highlight java %}
	translator.addCustomHeaderFile("LiquidCrystal_I2C.h");
{% endhighlight %}

5. Import your header files and cpp file to eclipse. You should keep them well in your classpath.
6. Next you need to use file manipulation to read from your jar file, and write the file stream into the Aruino temp project location. You don't want to write to the destination, since they would clobber the source file to import.
7. Find the Aruino `Editor` instance along the `Context` instance and open your temp header and/or c++ file.
8. Set the editor to the ino file, or else your translated text will be writing to the last file you opened by default.

{% highlight java %}
Sketch sketch = Context.getContext().getEditor().getSketch();
File folder = sketch.getFolder().getParentFile();
File header = new File(folder.getAbsolutePath() + File.separator + fileName);
InputStream is = getClass().getResource("/com/ardublock/translator/block/dfrobot/" + fileName).openStream();
OutputStream os = new FileOutputStream(header);
byte[] buffer = new byte[4096];
int bytesRead;
while ((bytesRead = is.read(buffer)) != -1) {
	os.write(buffer, 0, bytesRead);
}
is.close();
os.close();
{% endhighlight %}


## Conclusion

You can easily add new module for **simple input and output**.
You have to drill down to the `Translator` core for advanced code generation, but it is not a very difficult task because the source code is quite well orgnized. Thumbs up to the [Arublock](http://blog.ardublock.com) team.

