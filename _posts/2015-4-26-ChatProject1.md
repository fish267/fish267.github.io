---
author: Fish
title: ChatProject---1
layout: post
cagetories: work 
tags: java
---
I have not installed the Chinese input method. It is really cool to write this blog in English, lol~
It is a simple chat project implemented by java. I followed the MSB java tutorial and I am here to say thank you to the graceful teather.


### 1. Init a chat window

The first version is about to init a GUI window, it is so simple that you could not close it on clicking the X button. Based on the package Frame.
{% highlight java %}
import java.awt.*;

public class ChatClient extends Frame{
    TextField tf = new TextField();
    TextArea ta = new TextArea();
    public static void main(String[] args) {
        ChatClient cc = new ChatClient();
        cc.launchFrame();
    }

    public void launchFrame () {
        this.setLocation(400, 300);
        this.setSize(300, 400);
        this.add(tf, BorderLayout.SOUTH);
        this.add(ta, BorderLayout.NORTH);
        pack();
        this.setVisible(true);
    }
}

{% endhighlight %}

### 2. Add a closing function.

Package java.awt.event.* should be imported. Then give the window a listener.
{% highlight java %}
import java.awt.*;
import java.event.*;

public class ChatClient extends Frame() {
    TextField tf = new TextField();
    TextArea  ta = new TextArea();

    public static void main(String[] args) {
        new ChatClient().launchFrame(); 
    }

    public void launchFrame() {
        this.setLocation(400, 300);
        this.setSize(300, 400);
        this.add(tf, BorderLayout.SOUTH);
        this.add(ta, BorderLayout.NORTH);
        pack();
        // add action listener
        this.addWindowListener(new WindowAdapter) {
            @override

                public void windowClosing(WindowEvent arg0) {
                    System.exit(0);
                }
        }
        this.setVisible(true);
    }
}
{% endhighlight %}

### 3. Make textarea showing the content.

We are still on the beginning, and we try to let textare show what we typed int the textField.

{% highlight java %}
import java.awt.*;
import java.awt.event.*;

public class ChatClient() extends Frame {

    TextField tf = new TextField();
    TextArea ta = new TextArea();

    public static void main(String[] args) {
        new ChatClient().launchFrame();
    }

    public void launchFrame() {
        this.setLocation(400, 300);
        this.setSize(300, 300);
        this.add(tf, BorderLayout.SOUTH);
        this.add(ta, BorderLayout.NORTH);
        pack();
        this.addWindowListener(new WindowAdapter() {

            public void windowClosing(WindowEvent args) {
                System.exit(0);
            }
        });
        tf.addActionListener(new TFListener());
        this.setVisible(true);
    }

    private class TFListener implements ActionListener {

        public void actionPerformed(ActionEvent e) {
            String str = tf.getText().trim();
            ta.setText(str);
            tf.setText("");
        }
    }

}
{% endhighlight %}


### 4. Start a server

In last version we created a simple chat client which can be closed and can show the content we typed in. Now let us start to create the server based on java.net.*.

{% highlight java %}

import java.net.*;
import java.io.IOException;

public class ChatServer {

    public static void main(String[] args) {

        try {
            ServerSocket ss = new ServerSocket(8888);
            while(true) {
                Socket s =  ss.accept();
                System.out.println("A client has connected");
            }
         } catch (IOException e) {
                e.printStackTrace();
         }
        
    }
}

{% endhighlight %}


### 5. Let chatclient connect to chatserver

Write a ClientSocket which can connect to ServerSocket.

{% highlight java %}
import java.awt.*;
import java.awt.event.*;
import java.io.IOException;
import java.net.*;

public class ChatClient extends Frame {

    TextField tf = new TextField();
    TextArea  ta = new TextArea();

    public static void main(String[] args) {

        new ChatClient().launchFrame();
    }

    public void launchFrame() {
        
        this.setLocation(400, 300);
        this.setSize(300, 300);
        this.add(tf, BorderLayout.SOUTH);
        this.add(ta, BorderLayout.NORTH);
        this.addWindowListener(new WindowAdapter() {

            public void windowClosing(WindowEvent arg) {

                System.exit(0);
            }
        });

        tf.addActionListener(new TFListener());
        this.setVisible(true);
        connect();
    }

    private class TFListener implements ActionListener {

        public void  actionPerformed(ActionEvent e) {
            String str = tf.getText().trim();
            ta.setText(str);
            tf.setText("");
        }
    }

    public void connect {
        try{ 
            Socket s = new Sokcet("127.0.0.1", 8888);
            System.out.println("Connected to server!");
        } catch(IOException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}
