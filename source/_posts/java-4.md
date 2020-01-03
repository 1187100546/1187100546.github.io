---
title: java第四次实验作业
date: 2020-01-03 16:11:36
tags: [java]
categories: java
---

java第四次实验作业。

<!--more-->

### 一、

 编写一个多线程程序，在其中实现两个定时线程，一个线程每隔1秒显示一次秒数，另一个每隔3秒显示一次字母(a,b,…)。  

```java
package hw6;

public class nine_2 extends Thread {
	String str;
	public nine_2(String str)
	{
		this.str=str;
	}
	public void run()
	{
		int i=0;
		if(this.str.equals("a"))
		{
			while(true)
			{
				i++;
				System.out.println(i);
				try {
					sleep(1000);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
		else
		{
			int j=0;
			while(true)
			{
				System.out.println((char)(97+j%26));
				j++;
				try {
					sleep(3000);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
	public static void main(String[] args)
	{
		nine_2 thread1=new nine_2("a");
		nine_2 thread2=new nine_2("b");
		thread1.start();
		thread2.start();
		
	}

}

```

![laPBsH.png](https://s2.ax1x.com/2020/01/03/laPBsH.png)

### 二、

编写一个图形用户界面程序，窗体的宽度300，高度150，布局管理器为null，窗体上有二个标签和二个按钮，标签的位置为（10,30）和（200,60），按钮的位置为（50,100）和（150,100），它们的宽度和高度都是80和20。编写一个线程，该线程可以让标签向右或向左移动10次，每次移动10个单位，间隔1秒，通过按钮的动作事件启动上述线程，“向右走”按钮启动“向右移标签”，“向左走”按钮启动“向左移标签”，界面如下图所示。  

![laP6ot.png](https://s2.ax1x.com/2020/01/03/laP6ot.png)

```java
package hw6;

import java.awt.BorderLayout;
import java.awt.EventQueue;
import java.awt.Point;

import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.border.EmptyBorder;

import hw6.youyi.mythread;

import javax.swing.JLabel;
import javax.swing.JButton;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;

public class zuoyouyi extends JFrame {

	private JPanel contentPane;
	private JLabel lblNewLabel;
	private JLabel lblNewLabel_1;

	/**
	 * Launch the application.
	 */
	public static void main(String[] args) {
		EventQueue.invokeLater(new Runnable() {
			public void run() {
				try {
					zuoyouyi frame = new zuoyouyi();
					frame.setVisible(true);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}
	class mythread extends Thread {
		String str;
		public mythread(String str)
		{
			this.str=str;
		}
		public void run()
		{
			if(this.str.equals("right"))
			{
				Point p=lblNewLabel.getLocation();
				int i=1;
				while(i<=10)
				{
					p.x=p.x+10;
					lblNewLabel.setLocation(p);
					try {
						sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					i++;
				}
			}
			if(this.str.equals("left"))
			{
				Point p=lblNewLabel_1.getLocation();
				int i=1;
				while(i<=10)
				{
					p.x=p.x-10;
					lblNewLabel_1.setLocation(p);
					try {
						sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					i++;
				}
			}
		}
	}
	/**
	 * Create the frame.
	 */
	public zuoyouyi() {
		setTitle("\u5DE6\u53F3\u6587\u5B57\u79FB\u52A8");
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		setBounds(100, 100, 450, 300);
		contentPane = new JPanel();
		contentPane.setBorder(new EmptyBorder(5, 5, 5, 5));
		setContentPane(contentPane);
		contentPane.setLayout(null);
		
		lblNewLabel = new JLabel("\u5411\u53F3\u79FB\u6807\u7B7E");
		lblNewLabel.setBounds(42, 43, 76, 16);
		contentPane.add(lblNewLabel);
		
		lblNewLabel_1 = new JLabel("\u5411\u5DE6\u79FB\u6807\u7B7E");
		lblNewLabel_1.setBounds(302, 99, 76, 16);
		contentPane.add(lblNewLabel_1);
		
		JButton btnNewButton = new JButton("\u5411\u53F3\u79FB");
		btnNewButton.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				mythread thread1=new mythread("right");
				thread1.start();
			}
		});
		btnNewButton.setBounds(69, 180, 103, 25);
		contentPane.add(btnNewButton);
		
		JButton btnNewButton_1 = new JButton("\u5411\u5DE6\u79FB");
		btnNewButton_1.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				mythread thread2=new mythread("left");
				thread2.start();
			}
		});
		btnNewButton_1.setBounds(247, 180, 103, 25);
		contentPane.add(btnNewButton_1);
	}

}

```

![laP4yQ.png](https://s2.ax1x.com/2020/01/03/laP4yQ.png)

### 三、

小球向右下角移动。

```java
package hw6;

import java.awt.BorderLayout;
import java.awt.EventQueue;
import java.awt.Graphics;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;

import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.border.EmptyBorder;

public class lizi extends JFrame {

	private JPanel contentPane;
	int i=0;

	/**
	 * Launch the application.
	 */
	public static void main(String[] args) {
		EventQueue.invokeLater(new Runnable() {
			public void run() {
				try {
					lizi frame = new lizi();
					frame.setVisible(true);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}
	class mythread extends Thread{
		public void run()
		{
			for( i=1;i<=5;i++)
			{
				repaint();
				try {
					sleep(1000);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
	public void paint(Graphics g) {
	    super.paint(g);
	    g.fillOval(50+i*10, 50+i*10, 100, 100);
	}

	/**
	 * Create the frame.
	 */
	public lizi() {
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		setBounds(100, 100, 450, 300);
		contentPane = new JPanel();
		contentPane.setBorder(new EmptyBorder(5, 5, 5, 5));
		contentPane.setLayout(new BorderLayout(0, 0));
		setContentPane(contentPane);
		this.addMouseListener(new MouseAdapter() {
			public void mouseClicked(MouseEvent e) {
				mythread thread1=new mythread();
				thread1.start();
			}
		});
			
		
	}

}

```

![laiiY6.png](https://s2.ax1x.com/2020/01/03/laiiY6.png)

### 四、

编程读取中国教育网主页html文档( URL：http://www.edu.cn )，显示该文档内容，并判断其中是否有字符串“中国教育和科研计算机网网络中心”。

  

```java
package hw6;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.net.MalformedURLException;
import java.net.URL;
public class shiyi_1 {
	public static void main(String[] args)
	{
		boolean flag=false;
		try {
			URL url=new URL("http://www.edu.cn");
			InputStream is=null;
			try {
				is = url.openStream();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		    InputStreamReader isr=null;
			try {
				isr = new InputStreamReader(is,"utf-8");
			} catch (UnsupportedEncodingException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		    BufferedReader br = new BufferedReader(isr);
		    try {
				String data=br.readLine();
				 while(data != null){
					 System.out.println(data);
					 if(data.contains("中国教育和科研计算机网网络中心"))
					 {
						 flag=true;
					 }
					 data = br.readLine();
				 }
				 br.close();
				 isr.close();
				 is.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		    
		} catch (MalformedURLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		if(flag==true)
		{
			System.out.println("含有");
		}
		else
		{
			System.out.println("不含有");
		}
	}
}

```

![laF4Vs.png](https://s2.ax1x.com/2020/01/03/laF4Vs.png)

### 五、

编写一个基于TCP的Socket程序，服务器向客户端发送一个字符串："Socket你好!"，客户端将接收到的字符串输出到屏幕。在一台PC上测试该程序。

编写一个基于TCP的Socket程序，服务器向客户端发送一个字符串："Socket你好!"，客户端将接收到的字符串输出到屏幕。在两台PC之间测试该程序。（本题上机时两人一组，一个作为服务方，另一个作为客户方，然后两人之间进行对话）

编写一个Socket网络通讯应用程序，实现如下功能：

 1）客户端能够发任意的信息给服务器端；

 2）服务器端将收到的客户端信息返还给客户端。

 把3个合在一起

服务器端：

```java
package hw6;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;

public class sever {
	public static void main(String[] args) {
        try {
            // 构造服务器ServerSocket对象
            ServerSocket ss = new ServerSocket(30102);
            System.out.println("服务器准备就绪！");
            
            while(true){
                Socket s = ss.accept();
                // 线程
                new ServerThread(s).start();
            }
        	} catch (IOException e) {
        		e.printStackTrace();
        	}
    }
}

/**
 * 服务器端与客户端会话的线程
 */
class ServerThread extends Thread {
    private Socket s = null;
    private BufferedReader read = null;
    private PrintStream print = null;
    public ServerThread(Socket s) {
        this.s = s;
        try {
            
            read = new BufferedReader(new InputStreamReader(s.getInputStream()));
            print = new PrintStream(s.getOutputStream());
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    /**
     * 线程的运行run方法
     */
    public void run() {
        try {
            String message = null;
            while (!(message = read.readLine()).equals("exit")){
                
            	System.out.println("client发送："+message);
                print.println("Socket你好!" + message);
            }
        } catch (IOException e) {
        } finally {
            
            try {
                
                if(!s.isClosed()){
                    
                    s.close();
                }
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        }
    }
}

```

客户端：

```java
package hw6;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.Socket;
import java.net.UnknownHostException;
import java.util.Scanner;

public class client {
	public static void main(String[] args) {
        try {
            // 构造与服务器通讯的Socket对象
            Socket s = new Socket("127.0.0.1", 30102);
            // 启动一个线程与服务器通讯
            new LinkThread(s).start();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
   
  
   
}
/**
 * 与服务器通讯的线程
 */
class LinkThread extends Thread {
    private Socket s = null;
    private PrintStream out = null;
    private BufferedReader in = null;
    private Scanner scanner = null;
   
    public LinkThread(Socket s) {
       
        this.s = s;
        try {
            
            out = new PrintStream(s.getOutputStream());
            in = new BufferedReader(new InputStreamReader(s.getInputStream()));
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
   
    /**
     * 线程的运行run方法
     */
    public void run() {
        scanner = new Scanner(System.in);
        System.out.println("提示：如果要结束本次会话，请输入“exit”指令！");
        try {
            
            while(true){
                System.out.print("请输入：");
                String message = scanner.nextLine();
                out.println(message);
                out.flush();
                String str = in.readLine();
                if(str != null){
                    System.out.println(str);
                }else{
                    System.out.println("本次会话结束！");
                    return;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if(!s.isClosed()){
                    s.close();
                }
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        }
    }
   
}

```

![lakCRK.png](https://s2.ax1x.com/2020/01/03/lakCRK.png)

