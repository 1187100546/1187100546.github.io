---
title: java实验二
date: 2019-11-26 20:09:34
tags: [java]
categories: java
---

java实验二。

<!--more-->

### 问题1、

 打印一个三角形的1~9的乘法表。

```java
package hw2;

public class chenfa {
	public static void main(String[] args)
	{
		System.out.print("*"+"\t");
		int i;
		for(i = 1;i <= 9;i++)
		{
			System.out.print(i+"\t");
		}
		System.out.println();
		int x,y;
		for(x = 1;x <= 9;x++)
		{
			System.out.print(x+"\t");
			for(y = 1;y <= x; y++)
			{
				System.out.print(x*y+"\t");
			}
			System.out.println();
		}
	}
}
```

![QSrMrt.png](https://s2.ax1x.com/2019/11/26/QSrMrt.png)

### 问题2、

编写一程序，将从键盘输入的每个月份数(整数)显示出其对应的英文，直至输入0结束，注意对非法数据的处理。  (while,switch语句)

```java
package hw2;
import java.util.*;
public class two {
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		System.out.println("请输入月份");
		int m;
		while(true)
		{
			m=s.nextInt();
			if(m==0)
			{
				System.out.println("输入结束");
				break;
			}
			switch(m)
			{
			case 1:System.out.println("January");break;
			case 2:System.out.println("February");break;
			case 3:System.out.println("March");break;
			case 4:System.out.println("April");break;
			case 5:System.out.println("May");break;
			case 6:System.out.println("June");break;
			case 7:System.out.println("July");break;
			case 8:System.out.println("August");break;
			case 9:System.out.println("September");break;
			case 10:System.out.println("October");break;
			case 11:System.out.println("November");break;
			case 12:System.out.println("December");break;
			default:System.out.println("非法字符");break;
			
			}
		}
		
		
	}
}
```

![QSrJPg.png](https://s2.ax1x.com/2019/11/26/QSrJPg.png)

### 问题3、

打印图案：一个由n行星花组成的三角形。如n=5时的图案为：

```java
package hw2;
import java.util.*;
public class three {
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		int n;
		System.out.println("请输入行数");
		n=s.nextInt();
		int x,y,z;
		for(x=1;x<=n;x++)
		{
			for(y=1;y<=n-x;y++)
			{
				System.out.print(" ");
			}
			for(z=1;z<=x;z++)
			{
				System.out.print("* ");
			}
			System.out.println();
		}
	}
}
```

![QSrrIU.png](https://s2.ax1x.com/2019/11/26/QSrrIU.png)

### 问题4、

打印出所有的“水仙花数”。所谓“水仙花数”是指一个三位数，其各位数字的立方和等于该数本身。例如153是一个“水仙花数”，因为153=13+53+33。

```java
package hw2;
import java.util.*;
public class four {
	public static void main(String[] args)
	{
		//Scanner s=new Scanner(System.in);
		System.out.println("水仙花数为：");
		int x;
		for(x=100;x<=999;x++)
		{
			double sum;
			sum=Math.pow(x/100,3)+Math.pow((x%100)/10,3)+Math.pow(x%10,3);
			if(sum==x)
			{
				System.out.println(x);
			}
		}
	}
}
```

![QSrveP.png](https://s2.ax1x.com/2019/11/26/QSrveP.png)

### 问题5、

编写一个程序，从键盘读一个年份的数字，然后判断该年是否是闰年，如果是就输出“闰年”，如果不是就输出“非闰年”。

```java
package hw2;
import java.util.*;
public class five {
	static boolean panduan(int x)
	{
		if(x%400==0)
			return true;
		if(x%420==0&&x%100!=0)
			return true;
		return false;
	}
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		System.out.println("请输入年份");
		int year;
		year=s.nextInt();
		if(panduan(year))
		{
			System.out.println(year+"年 是闰年");
		}
		else
		{
			System.out.println(year+"年 是非闰年");
		}
	}
}
```

![QSskyn.png](https://s2.ax1x.com/2019/11/26/QSskyn.png)

### 问题6、

统计个位数是6，并且能被3整除的五位数共有多少个。

```java
package hw2;

public class six {
	public static void main(String[] args)
	{
		int x;
		int sum=0;
		for(x=10000;x<=99999;x++)
		{
			if(x%10==6&&x%3==0)
			{
				sum++;
			}
		}
		System.out.println("个位数是6，并且能被3整除的五位数共有"+sum+"个");
	}
}
```

![QSsKW4.png](https://s2.ax1x.com/2019/11/26/QSsKW4.png)

### 问题7、

编写一个程序，在其中建立一个有10个整数的数组，运行后从键盘输入10个数，然后输出其中的最小数。

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
}
```

![QSsB6A.png](https://s2.ax1x.com/2019/11/26/QSsB6A.png)

### 问题8、

编写一个程序，在其中定义一个6´6的二维整型数组, 利用随机函数产生36个10~20之间的随机整数放入，然后将数组输出到屏幕上(6行6列格式)。最后计算出数组中对角线元素的平方根和。

```java
package hw2;
import java.util.*;
public class erweishuzu {
	public static void main(String[] args)
	{
		int a[][]=new int[6][6];
		int i,j;
		for(i=0;i<6;i++)
		{
			for(j=0;j<6;j++)
			{
				a[i][j]=(int)(10+Math.random()*10);
				System.out.print(a[i][j]+" ");
			}
			System.out.println();
		}
		double sum=0;
		int l;
		for(l=0;l<6;l++)
		{
			sum=sum+Math.sqrt(a[l][l])+Math.sqrt(a[5-l][l]);
		}
		System.out.println("平方根和为："+sum);
	}
}
```

![QS6Qq1.png](https://s2.ax1x.com/2019/11/26/QS6Qq1.png)

