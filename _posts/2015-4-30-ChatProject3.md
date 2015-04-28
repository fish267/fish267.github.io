---
author: Fish
layout: post
title: ChatProject3 
categories: work 
tags: java 
---
This is the final version of the chat project, here we modified the chatclient.java using multithread. 
<b>Chatlient.java</b>

{% highlight java %}

package Chat;

import java.awt.BorderLayout;
import java.awt.Frame;
import java.awt.TextArea;
import java.awt.TextField;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.EOFException;
import java.io.IOException;
import java.net.Socket;
import java.net.SocketException;
import java.net.UnknownHostException;

public class ChatClient extends Frame {
    Socket s = null;
    DataOutputStream dos = null;
    DataInputStream dis = null;
    private boolean bConnected = false;

    TextField tfTxt = new TextField();
    TextArea taContent = new TextArea();
    Thread tRecv = new Thread(new RecvThread());

    public static void main(String[] args) {
        new ChatClient().launchFrame();
    }

    public void launchFrame() {
        setLocation(400, 300);
        this.setSize(300, 300);
        add(tfTxt, BorderLayout.SOUTH);
        add(taContent, BorderLayout.NORTH);
        pack();
        this.addWindowListener(new WindowAdapter() {

            @Override
            public void windowClosing(WindowEvent arg0) {
                disconnect();
                System.exit(0);
            }

        });
        tfTxt.addActionListener(new TFListener());
        setVisible(true);
        connect();

        tRecv.start();
    }

    public void connect() {
        try {
            s = new Socket("127.0.0.1", 8888);
            dos = new DataOutputStream(s.getOutputStream());
            dis = new DataInputStream(s.getInputStream());
            System.out.println("connected!");
            bConnected = true;
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    public void disconnect() {
        try {
            dos.close();
            dis.close();
            s.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    private class TFListener implements ActionListener {

        @Override
        public void actionPerformed(ActionEvent e) {
            String str = tfTxt.getText().trim();
            // taContent.setText(str);
            tfTxt.setText("");

            try {
                // System.out.println(s);
                dos.writeUTF(str);
                dos.flush();
                // dos.close();
            } catch (IOException e1) {
                e1.printStackTrace();
            }

        }

    }

    private class RecvThread implements Runnable {

        @Override
        public void run() {
            try {
                while (bConnected) {
                    String str = dis.readUTF();
                    // System.out.println(str);
                    taContent.setText(taContent.getText() + str + '\n');
                }
            } catch (SocketException e) {
                System.out.println("退出了，bye!");
            } catch (EOFException e) {
                System.out.println("推出了，bye - bye!");
            } catch (IOException e) {
                e.printStackTrace();
            }

        }

    }
}
{% endhighlight %}
<!--more-->

##<b>ChatServer.java</b>

{% highlight java %}
package Chat;

import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.EOFException;
import java.io.IOException;
import java.net.BindException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;
import java.util.List;

public class ChatServer {
    boolean started = false;
    ServerSocket ss = null;

    List<Client> clients = new ArrayList<Client>();

    public static void main(String[] args) {
        new ChatServer().start();
    }

    public void start() {
        try {
            ss = new ServerSocket(8888);
            started = true;
        } catch (BindException e) {
            System.out.println("端口使用中....");
            System.out.println("请关掉相关程序并重新运行服务器！");
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
                clients.add(c);
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
        private DataOutputStream dos = null;
        private boolean bConnected = false;

        public Client(Socket s) {
            this.s = s;
            try {
                dis = new DataInputStream(s.getInputStream());
                dos = new DataOutputStream(s.getOutputStream());
                bConnected = true;
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        public void send(String str) {
            try {
                dos.writeUTF(str);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void run() {
            try {
                while (bConnected) {
                    String str = dis.readUTF();
                    System.out.println(str);
                    for (int i = 0; i < clients.size(); i++) {
                        Client c = clients.get(i);
                        c.send(str);
                    }
                }
            } catch (EOFException e) {
                System.out.println("Client closed!");
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    if (dis != null)
                        dis.close();
                    if (dos != null)
                        dos.close();
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

#END
