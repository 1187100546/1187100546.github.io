---
title: java第二次实验作业
date: 2019-12-30 13:15:57
tags: [java]
categories: java
---

Java第二次实验作业。

<!--more-->

### 一、

编写一个程序，其中设计一个矩阵类Matrix

Matrix类：

```java
package hw4;

public class Matrix {
	private int m,n;
    protected int[][]ma;
    
    public Matrix(int m,int n) {
        this.m = m;
        this.n = n;
    }
    public void setMa(int[][]ma) {
        this.ma = ma;
    }

    public Matrix cheng(Matrix a) {
        if(n != a.m) return null;
        Matrix ans= new Matrix(m,a.n);
        int ansm = m,ansn = a.n;
        int[][]ansa = new int[ansm][ansn];
        for(int i = 0; i < ansm; i++) {
            for(int j = 0; j < ansn; j++) {
                for(int k = 0; k < n; k++) {
                    ansa[i][j] += ma[i][k]*a.ma[k][j];  
                }
            }
        }
        ans.ma = ansa;
        return ans;
    }

    public void print() {
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                System.out.print(ma[i][j]+" ");
            }
            System.out.println();
        }
    }
}

```

Main类：

```java
package hw4;
import java.util.*;
public class main_matrix {
	public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入矩阵A的行列数：");
        int am = scanner.nextInt();
        int an = scanner.nextInt();
        System.out.println("请输入矩阵A：");
        Matrix matrixA = new Matrix(am, an);
        int[][]ma = new int[am][an];
        for(int i = 0; i < am; i++) {
            for(int j = 0; j < an; j++) {
                ma[i][j] = scanner.nextInt(); 
            }
        }
        matrixA.setMa(ma);
        System.out.println("请输入矩阵B的行列数：");
        int bm = scanner.nextInt();
        int bn = scanner.nextInt();
        System.out.println("请输入矩阵B：");
        Matrix matrixB = new Matrix(bm, bn);
        int[][]mb = new int[bm][bn];
        for(int i = 0; i < bm; i++) {
            for(int j = 0; j < bn; j++) {
                mb[i][j] = scanner.nextInt(); 
            }
        }
        matrixB.setMa(mb);
        //计算矩阵A乘矩阵B并打印结果
        System.out.println("A*B的结果为：");
        matrixA.cheng(matrixB).print();
        scanner.close();
    }
}

```

![lUxlMn.png](https://s2.ax1x.com/2020/01/03/lUxlMn.png)

### 二、

设计一个程序，其中含有一个接口Shape(形状)，其中有求形状的面积的方法area()。再定义三个实现接口的类：三角型类、矩形类和圆类。在主方法中创建Shape类型的一维数组，它有三个元素，放置三个对象，分别表示三角形、矩形和圆，然后利用循环输出三个图形的面积。

Shape接口：

```java
package hw4;

public interface shape {
	double area();
}

```

Circle类：

```java
package hw4;

public class circle implements shape {
	final float pi=3.14f;
	float r;
	circle()
	{
		
	}
	circle(float r)
	{
		this.r=r;
	}
	public double area()
	{
		return pi*r*r;
	}
}

```

Rectangle类：

```java
package hw4;

public class rectangle implements shape{
	float w;
	float h;
	rectangle()
	{
		
	}
	rectangle(float w,float h)
	{
		this.w=w;
		this.h=h;
	}
	public double area()
	{
		return w*h;
	}
}

```

Square类：

```java
package hw4;
import java.util.*;
public class square implements shape{
	float a,b,c;
	square()
	{
		
	}
	square(float a,float b,float c)
	{
		this.a=a;
		this.b=b;
		this.c=c;
	}
	public double area()
	{
		float p=(a+b+c)/2;
		return Math.sqrt(p*(p-a)*(p-b)*(p-c));
	}
}

```

主类：

```java
package hw4;
import java.util.*;
public class main_shape {
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		shape shape[]=new shape[3];
		int i;
		for(i=0;i<3;i++)
		{
			System.out.println("请选择输入的图形：1、圆形，2、矩形，3、三角形：");
			int x=s.nextInt();
			if(x==1)
			{
				System.out.println("请输入圆形的半径：");
				shape[i]=new circle(s.nextFloat());
				System.out.println("面积是："+shape[i].area());
			}
			else if(x==2)
			{
				System.out.println("请输入矩形宽和高：");
				shape[i]=new rectangle(s.nextFloat(),s.nextFloat());
				System.out.println("面积是："+shape[i].area());
			}
			else if(x==3)
			{
				System.out.println("请输入三角形a,b,c:");
				shape[i]=new square(s.nextFloat(),s.nextFloat(),s.nextFloat());
				System.out.println("面积是："+shape[i].area());
			}
			else
			{
				System.out.println("非法输入，程序退出");
				break;
			}
				
		}
	}
}

```

![lUxjQs.png](https://s2.ax1x.com/2020/01/03/lUxjQs.png)

### 三、

编一程序，在其中定义一个代表篮球队的类，它有放置队员姓名的向量并放入队员的姓名，再写两个方法：

1）在向量中查找某人。若找到则输出“找到此人！”，否则输出“查无此人！”。

2）删除队员。先查找该人，若找到则删除，否则输出“无此队员！”。

```java
package hw4;

public class basketball {
	String dy[]=new String[10];
	basketball()
	{
		int i;
		for(i=0;i<10;i++)
		{
			dy[i]=null;
		}
	}
	public void insert(String name)
	{
		int i;
		for(i=0;i<10;i++)
		{
			if(dy[i]==null)
			{
				dy[i]=name;
				System.out.println("插入成功");
				break;
			}
		}
	}
	public void delete(String name)
	{
		int i;
		for(i=0;i<10;i++)
		{
			if(dy[i].equals(name))
			{
				dy[i]=null;
				System.out.println("删除成功");
				break;
			}
		}
		if(i==10)
		{
			System.out.println("查无此人");
		}
	}
	public void show()
	{
		int i;
		for(i=0;i<10;i++)
		{
			System.out.print(dy[i]+",");
		}
	}
}

```

主类：

```java
package hw4;
import java.util.*;
public class main_basket {
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		basketball b=new basketball();
		while(true)
		{
			System.out.println("输入1插入，2删除，其它退出");
			int x=s.nextInt();
			if(x==1)
			{
				b.insert(s.nextLine());
				b.show();
			}
			else if(x==2)
			{
				b.delete(s.nextLine());
				b.show();
			}
			else
			{
				b.show();
				System.out.println("程序已退出");
				break;
			}
		}
	}
}

```

![lUzEl9.png](https://s2.ax1x.com/2020/01/03/lUzEl9.png)

### 四、

设计一个继承Vector类的队列类Queue，实现队列的先进先出功能, 类中含有两个方法：入队inqueue和出队outqueue(要充分利用Vector类的方法)。在主方法中创建一个队列类对象，然后依次完成 “111” 入队、“222”入队、出队一元素(输出到屏幕)、“333”入队，最后出队所有元素并且输出到屏幕。  

```java
package hw4;
import java.util.*;
public class queue extends Vector{
	public void inqueue(Object obj)
	{
		addElement(obj);
	}
	public Object outqueue()
	{
		Object b=firstElement();
		removeElementAt(0);
		return b;
	}
	public static void main(String[] args)
	{
		queue q1=new queue();
		q1.inqueue("111");
		q1.inqueue("222");
		Object bb=q1.outqueue();
		System.out.println(bb);
		q1.inqueue("333");
		bb=q1.outqueue();
		System.out.println(bb);
		bb=q1.outqueue();
		System.out.println(bb);
	}
	
}

```

![laSCnI.png](https://s2.ax1x.com/2020/01/03/laSCnI.png)

### 五、

java.util包中有个类“Arrays”，它有个方法“sort(<数组名>)”，功能是将数组按升序排序。请编一程序，在其中创建一个数组，然后利用sort方法进行排序。  

```java
package hw2;
import java.util.*;
public class seven {
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		System.out.println("请输入10个数");
		int a[]=new int[10];
		int i;
		for(i=0;i<10;i++)
		{
			a[i]=s.nextInt();
		}
		Arrays.sort(a);
		for(i=0;i<10;i++)
		{
			System.out.print(a[i]+"\t");
		}
	}

```

![laSMBq.png](https://s2.ax1x.com/2020/01/03/laSMBq.png)

### 六、

编写一个让小朋友做十次加法的程序，要求程序中生成两个不大于50的随机正整数a和b，其中a由Math类的随机函数生成，b则利用机器当前时间的秒数和分数生成，在小朋友回答后要给出对错的判断。  

```java
package hw4;
import java.time.LocalDateTime;
import java.util.*;
public class jisuanti {
	public static void main(String[] args)
	{
		int i=0;
		while(i<10)
		{
			Scanner s=new Scanner(System.in);
			LocalDateTime time=LocalDateTime.now();
			int a=(int)(Math.random()*50);
			int b=(time.getMinute()+time.getSecond())%50;
			System.out.println(a+"+"+b+"=");
			if(s.nextInt()==(a+b))
			{
				System.out.println("yes");
			}
			else
			{
				System.out.println("no");
			}
		}
	}
}

```

![lapZa6.png](https://s2.ax1x.com/2020/01/03/lapZa6.png)

