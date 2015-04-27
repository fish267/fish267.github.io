---
author: Fish
layout: post
title:  ChatProject---2 
categories: java 
tags: learning 
---

###6. ChatClient send message to ChatServer.

In this version, we try to make the communications between chatclient and chatserver.

<b>ChatClient</b>

{% highlight java %}
package Chat;

import java.io.DataOutputStream;
import java.io.IOException;
import java.net.*;
import java.awt.*;
import java.awt.event.*;

public class ChatClient extends Frame {
    TextField tf = new TextField();
    TextArea ta = new TextArea();
    Socket s = null;

    public static void main(String[] args) {
        new ChatClient().launchFrame();
    }

    private void launchFrame() {
        this.setLocation(300, 500);
        this.setSize(300, 300);
        this.add(tf, BorderLayout.SOUTH);
        this.add(ta, BorderLayout.NORTH);
        this.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                System.exit(0);
                ;
            }
        });
        tf.addActionListener(new TFListener());
        this.pack();
        this.setVisible(true);
        connect();
    }

    private void connect() {
        try {
            s = new Socket("127.0.0.1", 8888);
            System.out.println("Conencted to server!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private class TFListener implements ActionListener {
        public void actionPerformed(ActionEvent e) {
            String str = tf.getText().trim();
            ta.setText(str);
            tf.setText("");
            // Send message
            try {
                DataOutputStream dos = new DataOutputStream(s.getOutputStream());
                dos.writeUTF(str);
                dos.flush();
                dos.close();
            } catch (IOException ee) {
                ee.printStackTrace();
            }
        }
    }
}
{% endhighlight %}
<!--more-->
<hr>
<b>ChatServer.java</B>

{% highlight java %}
package Chat;

import java.net.*;
import java.io.DataInputStream;
import java.io.IOException;

public class ChatServer {

    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(8888);
            while (true) {
                Socket s = ss.accept();
                System.out.println("A client has connected!");
                DataInputStream dis = new DataInputStream(s.getInputStream());
                String str = dis.readUTF();
                System.out.println(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

{% endhighlight %}


### 7. Send more message to chatserver.

There is an obvious bug in version6.0 that the chatclient could only send one message toserver. Beacuse the 'dos.close()' locked the progress. In this version we manage to fix it.

<b> ChatClient.java </b>
{% highlight java %}
package Chat;

import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.net.*;

public class ChatClient extends Frame {
    TextField tf = new TextField();
    TextArea ta = new TextArea();
    Socket s = null;
    DataOutputStream dos = null;

    public static void main(String[] args) {
        new ChatClient().launchFrame();
    }

    public void launchFrame() {
        this.setLocation(300, 500);
        this.setSize(300, 300);
        this.add(tf, BorderLayout.SOUTH);
        this.add(ta, BorderLayout.NORTH);
        this.addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                disconnect();
                System.exit(0);
            }
        });
        tf.addActionListener(new TFListener());
        this.pack();
        this.setVisible(true);
        connect();
    }

    public void connect() {
        try {
            s = new Socket("127.0.0.1", 8888);
            dos = new DataOutputStream(s.getOutputStream());
            System.out.println("Conencted to server!");
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void disconnect() {
        try {
            dos.close();
            s.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private class TFListener implements ActionListener {
        public void actionPerformed(ActionEvent e) {
            String str = tf.getText().trim();
            ta.setText(str);
            tf.setText("");
            // Send message
            try {
                dos.writeUTF(str);
                dos.flush();
            } catch (IOException ee) {
                ee.printStackTrace();
            }
        }
    }
}
{% highlight %}

<b>ChatServer.java </b>
{% highlight java %}
package Chat;

import java.net.*;
import java.io.DataInputStream;
import java.io.IOException;

public class ChatServer {

    public static void main(String[] args) {
        boolean started = false;
        DataInputStream dis = null;
        try {
            ServerSocket ss = new ServerSocket(8888);
            while (true) {
                Socket s = ss.accept();
                System.out.println("A client has connected!");
                started = true;
                while (started) {
                    dis = new DataInputStream(s.getInputStream());
                    String str = dis.readUTF();
                    System.out.println(str);
                }
                dis.close();
            }
        } catch (BindException e1) {
            System.out.println("The port has been used !");
            System.exit(0);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

{% endhighlight %} 


### 8. Allow one more client.

Finally we could talk to server via chatclient, some bugs such as "Closing windows causes exception" are omitted. It is a pitty that only one client is allowd to chat. In this version we gonna fix it based on multiThread.

<b>ChatSever.java </b>
{% highlight java }%
package Chat;

import java.io.*;
import java.net.*;

public class ChatServer {
    boolean started = false;
    ServerSocket ss = null;

    public static void main(String[] args) {
        new ChatServer().start();
    }

    public void start() {
        try {
            ss = new ServerSocket(8888);
            started = true;
        } catch (BindException e) {
            System.out.println("端口使用中....");
            System.exit(0);
        } catch (IOException e) {
            e.printStackTrace();
        }

        try {
            while (started) {
                Socket s = ss.accept();
                Client c = new Client(s);
                System.out.println("a client connected!");
                new Thread(c).start();
                // dis.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                ss.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }

    class Client implements Runnable {
        private Socket s;
        private DataInputStream dis = null;
        private boolean bConnected = false;

        public Client(Socket s) {
            this.s = s;
            try {
                dis = new DataInputStream(s.getInputStream());
                bConnected = true;
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }

        public void run() {
            try {
                while (bConnected) {
                    String str = dis.readUTF();
                    System.out.println(str);
                }
            } catch (EOFException e) {
                System.out.println("Client closed!");
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    if (dis != null)
                        dis.close();
                    if (s != null)
                        s.close();
                } catch (IOException e1) {
                    e1.printStackTrace();
                }
            }
        }

    }
}
{% endhighlight %}


### 9.  
