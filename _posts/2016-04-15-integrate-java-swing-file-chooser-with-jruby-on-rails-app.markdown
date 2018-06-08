---
layout: post
title: Java Swing  File Chooser for JRuby on Rails App
image: "/content/images/2016/04/jruby-duke-painting-at-lunar-logic-polska-5.jpg"
date: '2016-04-15 16:22:23'
tags:
- java
---

This blog post is based on a experimentation I did to integrate [Java Swing](//en.wikipedia.org/wiki/Swing_(Java)) file chooser on a [JRuby on Rails](//github.com/jruby/jruby/wiki/JRubyOnRails) backed web application. Let me warn you, this a dirty hack, wanted to see if the idea would work, not at all recommended for production use. Did this out of curiosity!

Security feature implemented in the web browsers denies the access of real local path of files selected by [HTML file](//www.ietf.org/rfc/rfc1867.txt) upload control. A fake path will be returned instead of the accurate path. This was the motivation to try something out of the box.

#### 1. Overview
Started out by creating a simple file chooser written in [JRuby](//jruby.org/). Invoked this program using [Ajax](//en.wikipedia.org/wiki/Ajax_(programming)) call. Bingo, the file chooser appeared up on the screen. Using this method were able to retrieve the local file path.
 
#### 2. The *FolderChooser.rb*
Create a JRuby program, import [JFrame](//docs.oracle.com/javase/8/docs/api/javax/swing/JFrame.html) and [JFileChooser](//docs.oracle.com/javase/8/docs/api/javax/swing/JFileChooser.html) classes from the javax.swing package. Define a method named initUI, inside it make use of the JFrame methods to display the file chooser on top of the window. Place this program inside the *lib* directory of the Rails application.

 
{% highlight ruby %}
include Java
import javax.swing.JFrame
import javax.swing.JFileChooser

class FolderChooser < JFrame
	
	def initUI

        self.setLocationRelativeTo(nil)
		self.setAlwaysOnTop(true)
        self.setLocationByPlatform(true)
		self.setUndecorated(true)
        self.setVisible(true)
		
		chooser = JFileChooser.new  
	    chooser.set_file_selection_mode(JFileChooser::DIRECTORIES_ONLY)
	    return_val = chooser.show_open_dialog(self)

	    if return_val == JFileChooser::APPROVE_OPTION
		  dir_path = chooser.get_selected_file.get_path
          self.dispose()
		  return dir_path

{% endhighlight %}


#### 3. Method to handle Ajax request
Inside the controller class create a method which handles the Ajax request and invokes the above JRuby program.
{% highlight ruby %}

require 'FolderChooser'

  def invoke_folder_chooser
    if request.xhr?
      folder_path = FolderChooser.new.initUI
      puts folder_path
      render :json => {
                 :path => folder_path
             }
    end
  end
{% endhighlight %}

#### 4. Route the Ajax request to the controller method
Define a route in the *config/routes.rb* file to match *invoke\_chooser* path to the *invoke\_folder\_chooser* action of *PsrOpController*.
{% highlight ruby %}
post '/invoke_chooser', :to=>'psr_op#invoke_folder_chooser'
{% endhighlight %}

#### 5. Ajax call to invoke the program
Create a JavaScript function which performs an Ajax request using [jQuery.ajax()](//api.jquery.com/jquery.ajax/) function.
{% highlight ruby %}
function swingChooser(){
    $.ajax({
        url: "invoke_chooser", // Route to the PsrOp Controller method
        type: "POST",
        dataType: "json",
        success: function(data) {
            console.log(data.path);
        },
        error: function(e) {
            console.log("Ajax error!")
            console.log(e)
        }
    });

}
{% endhighlight %}

#### 6. Add a button on the view
On the *erb* file specify a button which calls *swingChooser* JavaScript method on click.
{% highlight ruby %}
 <input type="button" value="Select Folder" onclick="swingChooser()"/>
{% endhighlight %}

#### 7. Conclusion
In this post we explored a way to integrate Java Swing to a Jruby on Rails app to access the local file path.

The **full implementation** of this post can be found in the [GitHub repository](//github.com/pranavek/blog/tree/master/2016/4/SwingFilechooser).
