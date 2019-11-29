---
title: java实验三
date: 2019-11-26 20:09:39
tags: [java]
categories: java
---

Java实验三。

<!--more-->

### 问题1、

编一程序，求两个正整数m、n的最大公约数。要求程序中有两个方法，分别使用循环和递归求最大公约数，最后在主方法中分别调用这两个方法求解56与91的最大公约数。

```java
package hw3;

public class one {
	public int cyc(int x,int y)
	{
		while(x%y!=0)
		{
			int s=x%y;
			x=y;
			y=s;
		}
		return y;
	}
	public int rec(int x,int y)
	{
		if(x%y==0)
			return y;
		else
			return rec(y,x%y);
	}
	public static void main(String[] args)
	{
		one o=new one();
		int m,n;
		m=56;
		n=91;
		System.out.println(o.cyc(m,n));
		System.out.println(o.rec(m,n));
	}
}
```

### 问题2、

编写一个完整的Java Application程序，其中设计一个复数类Complex，利用它验证两个复数 1+2i 和3+4i 相加产生一个新的复数 4+6i 。复数类Complex的设计必须满足如下要求：
1）Complex 的属性有：
realPart：int型，代表复数的实数部分；
maginPart：int型，代表复数的虚数部分。
2）Complex 的方法有：
Complex()：构造方法，将复数的实部和虚部都置0；
Complex(int r , int i )：构造方法，形参 r 为实部的初值，i为虚部的初值。
3）Complex complexAdd(Complex a)：将当前复数对象与形参复数对象相加，最后的结果仍是一个复数对象，返回给此方法的调用者。
4）String toString(): 把当前复数对象的实部、虚部组合成 a+bi 的字符串形式，其中a 和 b分别为实部和虚部的数据。

```Java
package hw3;

public class complex {
	int rp,mp;
	public complex()
	{
		rp=mp=0;
	}
	public complex(int i,int j)
	{
		rp=i;
		mp=j;
	}
	public complex complexadd(complex a)
	{
		this.rp=this.rp+a.rp;
		this.mp=this.mp+a.mp;
		return this;
	}
	public String toString()
	{
		String s=rp+"+"+mp+"i";
		return s;
	}
	public static void main(String[] args)
	{
		complex c1=new complex(2,3);
		complex c2=new complex(4,5);
		c1.complexadd(c2);
		System.out.println(c1.toString());
	}
}
```

### 问题3、

编写一个包含圆类的程序，并为圆类设计几个构造方法和一般方法，在主方法中创建一个圆类对象并输出它的周长和面积。
要求：
    属性有3个：x,y,r，分别放置圆心坐标和半径；
    构造方法有2个。一个是无参的，用于设置3个属性的值都为0；另一个有参的，用于设置3个属性的值，以确定一个具体的圆。
    计算周长的方法：double zc()；
计算面积的方法：double mj()。

```java
package hw3;
import java.util.*;
public class yuan {
	double x,y,r;
	public yuan()
	{
		x=y=r=0.0;
	}
	public yuan(double x,double y,double r)
	{
		this.x=x;
		this.y=y;
		this.r=r;
	}
	public double zc()
	{
		return 2*Math.PI*this.r;
	}
	public double mj()
	{
		return Math.PI*Math.pow(this.r, 2);
	}
}

```

### 问题4、

编写一个程序，它含有一个圆类、圆柱类和主类。
要求：
    1）圆类参考上一题中的圆类；
    2）圆柱类：继承圆类，并加入一个属性h(高)；
              构造方法(给4个属性赋值)；
              计算面积的方法(double mj())；
              计算体积的方法(double tj())。
       注意，要充分利用父类的方法。
    3）主类：在主方法中创建圆和圆柱类的对象，然后计算并输出它们的面积及圆柱的体积。

yuanzhu类

```java
package hw3;
import java.util.*;
public class yuanzhu extends yuan{
	double h;
	public yuanzhu(double x,double y,double r,double h)
	{
		super(x,y,r);
		this.h=h;
	}
	public double mj()
	{
		return super.zc()*this.h+2*super.mj();
	}
	public double tj()
	{
		return super.mj()*this.h;
	}
}

```

main类

```java
package hw3;

public class main_yz {
	public static void main(String[] args)
	{
		yuan yuan1=new yuan(2,3,2);
		yuanzhu yuanzhu1=new yuanzhu(4,5,6,2);
		System.out.println("圆的面积是："+yuan1.mj());
		System.out.println("圆柱的面积是："+yuanzhu1.mj());
		System.out.println("圆柱的体积是："+yuanzhu1.tj());
	}
}
```

![QScZwt.png](https://s2.ax1x.com/2019/11/26/QScZwt.png)

### 问题6、

编写一个含有5个类的程序：
      类Person: 
          属性：编号、姓名、性别；
          构造方法：确定编号和姓名；
          一般方法：修改编号、姓名，获取编号、姓名。
      类Teacher：继承类Person并增加：
          属性：系别；
          构造方法：调用父类的构造方法；
          一般方法：修改、获取系别。
      类Student：继承类Person并增加：
          属性：班级；
          构造方法：调用父类的构造方法；
          一般方法：修改、获取班级属性值。
      类Classes：
          属性：班级名称，学生名单(Student类对象的数组)；
          构造方法：确定班级名称；
          一般方法：建立学生名单，输出学生名单。
      类Main：
主类。主方法中创建一个班级，然后建立该班级的学生名单，最后 
输出学生名单。 

person类

```java
package hw3;

public class person {
	int id;
	String name,gender;
	public person()
	{
	}
	public person(int id,String name)
	{
		this.id=id;
		this.name=name;
	}
	public void revise_id(int id)
	{
		this.id=id;
	}
	public void revise_name(String name)
	{
		this.name=name;
	}
	public int get_id()
	{
		return this.id;
	}
	public String get_name()
	{
		return this.name;
	}
}

```

teacher类

```Java
package hw3;

public class teacher extends person {
	String faculty;
	public teacher()
	{
		
	}
	public teacher(int id,String name,String faculty)
	{
		super(id,name);
		this.faculty=faculty;
	}
	public void revise_facilty(String Faculty)
	{
		this.faculty=faculty;
	}
	public String get_faculty()
	{
		return this.faculty;
	}
}
```

student类

```Java
package hw3;

public class student extends person {
	String class_number;
	public student(int id,String name,String class_number)
	{
		super(id,name);
		this.class_number=class_number;
	}
	public void revise_class_number(String class_number)
	{
		this.class_number=class_number;
	}
	public String get_class_number()
	{
		return this.class_number;
	}
}

```

classes类

```Java
package hw3;

public class classes {
	String class_name;
	student student_list[];
	int student_number;
	teacher teacher;
	public classes(String class_name,teacher tec)
	{
		this.class_name=class_name;
		this.teacher=tec;
	}
	public void set_student_list(student student_list[],int student_number)
	{
		this.student_list=student_list;
		this.student_number=student_number;
	}
	public void print_student_list()
	{
		System.out.println("班级名称是："+"\t"+this.class_name);
		System.out.println("任课教师是："+"\t"+this.teacher.name);
		System.out.println("学生名单如下：");
		int i;
		for(i=0;i<this.student_number;i++)
		{
			System.out.println("\t"+"姓名："+"\t"+this.student_list[i].name+"\t"+"学号:"+"\t"+this.student_list[i].id);
		}
	}
}
```

main类

```Java
package hw3;
import java.util.*;
public class main_class {
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		System.out.println("请输入班级名称：");
		String cn=s.nextLine();
		System.out.println("请输入任课老师姓名和院系");
		teacher tec=new teacher();
		tec.revise_name(s.nextLine());
		tec.revise_facilty(s.nextLine());
		classes banji=new classes(cn,tec);
		System.out.println("请输入学生人数：");
		int n;
		n=s.nextInt();
		student stu[]=new student[n];
		int i;
		System.out.println("请输入学生学号和姓名:");
		for(i=0;i<n;i++)
		{
			stu[i]=new student(s.nextInt(),s.nextLine(),cn);
		}
		banji.set_student_list(stu, n);
		banji.print_student_list();
	}
}

```

![QSc77t.png](https://s2.ax1x.com/2019/11/26/QSc77t.png)

