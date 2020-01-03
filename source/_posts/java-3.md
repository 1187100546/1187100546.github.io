---
title: java第三次实验作业
date: 2020-01-03 16:02:06
tags: [java]
categories: java
---

Java第三次实验作业。

<!--more-->

### 一、

开发一个加、减、乘、除四则运算器。用户界面如下图：

![laCkAx.png](https://s2.ax1x.com/2020/01/03/laCkAx.png)

```java
package hw5;

import java.awt.*;
import java.awt.event.ActionListener;

import javax.swing.*;
import javax.swing.border.EmptyBorder;

import javafx.event.ActionEvent;


public class jsq extends JFrame {

	private JPanel contentPane;

	private JTextField textField;
	private JTextField textField_1;
	private JTextField textField_2;
	/**
	 * Launch the application.
	 */
	public static void main(String[] args) {
		EventQueue.invokeLater(new Runnable() {
			public void run() {
				try {
					jsq frame = new jsq();
					frame.setVisible(true);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}

	/**
	 * Create the frame.
	 */
	public jsq() {
		setFont(new Font("幼圆", Font.BOLD, 20));
		setForeground(new Color(153, 50, 204));
		setTitle("\u8BA1\u7B97\u5668");
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		setBounds(100, 100, 450, 300);
		contentPane = new JPanel();
		contentPane.setBorder(new EmptyBorder(5, 5, 5, 5));
		setContentPane(contentPane);
		contentPane.setLayout(null);
		
		textField = new JTextField();
		textField.setBounds(0, 0, 432, 56);
		contentPane.add(textField);
		textField.setColumns(10);
		
		textField_1 = new JTextField();
		textField_1.setBounds(0, 67, 432, 56);
		contentPane.add(textField_1);
		textField_1.setColumns(10);
		
		JButton btnNewButton = new JButton("+");
		btnNewButton.setFont(new Font("幼圆", Font.BOLD, 16));
		btnNewButton.setForeground(new Color(50, 205, 50));
		btnNewButton.addActionListener(new ActionListener() {
			@Override
			public void actionPerformed(java.awt.event.ActionEvent e) {
				// TODO Auto-generated method stub
				double x = Double.parseDouble(textField.getText());
                double y = Double.parseDouble(textField_1.getText());
                double z = x+y;
                if(z - (int)z <= 0.00001) textField_2.setText(Integer.toString((int)z));
                else textField_2.setText(Double.toString(z));
			}
		});
		btnNewButton.setBounds(0, 122, 103, 54);
		contentPane.add(btnNewButton);
		
		JButton btnNewButton_1 = new JButton("-");
		btnNewButton_1.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
			}

			@Override
			public void actionPerformed(java.awt.event.ActionEvent e) {
				// TODO Auto-generated method stub
				double x = Double.parseDouble(textField.getText());
                double y = Double.parseDouble(textField_1.getText());
                double z = x-y;
                if(z - (int)z <= 0.00001) textField_2.setText(Integer.toString((int)z));
                else textField_2.setText(Double.toString(z));
			}
		});
		btnNewButton_1.setFont(new Font("幼圆", Font.BOLD, 20));
		btnNewButton_1.setForeground(new Color(255, 69, 0));
		btnNewButton_1.setBounds(115, 122, 103, 54);
		contentPane.add(btnNewButton_1);
		
		JButton btnNewButton_2 = new JButton("*");
		btnNewButton_2.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
			}

			@Override
			public void actionPerformed(java.awt.event.ActionEvent e) {
				// TODO Auto-generated method stub
				double x = Double.parseDouble(textField.getText());
                double y = Double.parseDouble(textField_1.getText());
                double z = x*y;
                if(z - (int)z <= 0.00001) textField_2.setText(Integer.toString((int)z));
                else textField_2.setText(Double.toString(z));
			}
		});
		btnNewButton_2.setFont(new Font("幼圆", Font.BOLD, 20));
		btnNewButton_2.setForeground(new Color(255, 215, 0));
		btnNewButton_2.setBounds(218, 122, 103, 54);
		contentPane.add(btnNewButton_2);
		
		JButton btnNewButton_3 = new JButton("/");
		btnNewButton_3.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
			}

			@Override
			public void actionPerformed(java.awt.event.ActionEvent e) {
				// TODO Auto-generated method stub
				double x = Double.parseDouble(textField.getText());
                double y = Double.parseDouble(textField_1.getText());
                double z = x/y;
                if(z - (int)z <= 0.00001) textField_2.setText(Integer.toString((int)z));
                else textField_2.setText(Double.toString(z));
			}
		});
		btnNewButton_3.setFont(new Font("幼圆", Font.BOLD, 20));
		btnNewButton_3.setForeground(new Color(0, 191, 255));
		btnNewButton_3.setBounds(329, 122, 103, 54);
		contentPane.add(btnNewButton_3);
		
		textField_2 = new JTextField();
		textField_2.setBounds(0, 187, 442, 58);
		contentPane.add(textField_2);
		textField_2.setColumns(10);
	}

}

```

![laCr5V.png](https://s2.ax1x.com/2020/01/03/laCr5V.png)

### 二、

编写一个程序，实现文件内容拷贝，具体过程如下：

 1) 建一文件myfile1.txt，写入内容“I am a student.”；

 2) 打开文件myfile1.txt，读出内容放入字符数组中；

 3) 再建一文件myfile2.txt，将字符数组中内容写入；

 4) 打开文件myfile2.txt，读出内容输出到屏幕。

```java
package hw5;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class shi_1 {
	

	public static void main(String[] args)
	{
		String str=null;
		String str1=null;
		String name_1="F:/myfile1.txt";
		String name_2="F:/myfile2.txt";
		FileWriter w1=null;
		try {
			w1=new FileWriter(name_1,true);
			w1.write("I am a student.");
			w1.close();
			System.out.println("myfile1.txt写入成功");
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		FileReader r1=null;
		BufferedReader br1=null;
		try {
			r1=new FileReader(name_1);
			br1=new BufferedReader(r1);
			try {
				str=br1.readLine();
				//System.out.println(str);
				br1.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		FileWriter w2=null;
		BufferedWriter bw2=null;
		try {
			w2=new FileWriter(name_2,true);
			bw2=new BufferedWriter(w2);
			bw2.write(str);
			bw2.flush();
			w2.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		FileReader r2=null;
		BufferedReader br2=null;
		try {
			r2=new FileReader(name_2);
			br2=new BufferedReader(r2);
			try {
				str1=br2.readLine();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			System.out.println(str1);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
	
}

```

![laCI56.png](https://s2.ax1x.com/2020/01/03/laCI56.png)

### 三、

 编写一个窗口程序，界面如下图，窗口FlowLayout布局，宽240，高200；文本框宽10，文本区大小为5行25列。  

![laPAqs.png](https://s2.ax1x.com/2020/01/03/laPAqs.png)

要求实现下列功能：

1) 点击“打开文件”，则弹出打开文件对话框，可从(D盘根目录)中选择一个字符文件，然后将选择的文件名称显示在文本框中；

2) 点击“显示文件内容”，则读出打开的文件的内容，并将读取的内容显示在文本区中；

```java
package hw5;

import java.awt.BorderLayout;
import java.awt.EventQueue;

import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.border.EmptyBorder;
import javax.swing.JButton;
import javax.swing.JTextField;
import javax.swing.JTextArea;
import javax.swing.JList;
import javax.swing.JComboBox;
import javax.swing.JFileChooser;

import java.awt.Panel;
import java.awt.ScrollPane;
import java.awt.event.ActionListener;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.awt.event.ActionEvent;

public class wenjian extends JFrame {

	private JPanel contentPane;
	private JTextField textField;

	/**
	 * Launch the application.
	 */
	public static void main(String[] args) {
		EventQueue.invokeLater(new Runnable() {
			public void run() {
				try {
					wenjian frame = new wenjian();
					frame.setVisible(true);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}

	/**
	 * Create the frame.
	 */
	public wenjian() {
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		setBounds(100, 100, 450, 300);
		contentPane = new JPanel();
		contentPane.setBorder(new EmptyBorder(5, 5, 5, 5));
		setContentPane(contentPane);
		contentPane.setLayout(null);
		
		JButton btnNewButton = new JButton("\u6253\u5F00\u6587\u4EF6");
		btnNewButton.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent arg0) {
				JFileChooser chooser = new JFileChooser();
		        chooser.setFileSelectionMode(JFileChooser.FILES_AND_DIRECTORIES);
		        chooser.showDialog(new JLabel(), "选择");
		        File file = chooser.getSelectedFile();
		        textField.setText(file.getAbsoluteFile().toString());
			}
		});
		btnNewButton.setBounds(23, 11, 129, 42);
		contentPane.add(btnNewButton);
		
		textField = new JTextField();
		textField.setBounds(199, 21, 198, 22);
		contentPane.add(textField);
		textField.setColumns(10);
		
		JTextArea textArea = new JTextArea();
		textArea.setBounds(33, 68, 346, 112);
		contentPane.add(textArea);
		
		JButton btnNewButton_1 = new JButton("\u663E\u793A\u6587\u4EF6\u5185\u5BB9");
		btnNewButton_1.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				try {
					FileReader fr=new FileReader(textField.getText());
					BufferedReader br=new BufferedReader(fr);
					try {
						textArea.setText(br.readLine());
					} catch (IOException e1) {
						// TODO Auto-generated catch block
						e1.printStackTrace();
					}
				} catch (FileNotFoundException e1) {
					// TODO Auto-generated catch block
					e1.printStackTrace();
				}
			}
		});
		btnNewButton_1.setBounds(124, 192, 167, 41);
		contentPane.add(btnNewButton_1);
	}
}

```

![laPniV.png](https://s2.ax1x.com/2020/01/03/laPniV.png)

