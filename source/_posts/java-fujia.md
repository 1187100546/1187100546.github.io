---
title: java继承实现多态的例子
date: 2019-11-29 09:08:31
tags: [java]
categories: java
---

利用继承实现多态。

<!--more-->

写一个动物园程序，可以向里面添加动物，并显示是第几个该类型的动物，并统计动物园里共有多少动物。

animal类：

```java
package hw3;

public class animal {
	static int sum=0;
	public animal()
	{
		
	}
	public void show()
	{
		System.out.println("动物园一共有"+sum+"只动物");
	}
	
}

```

dog类

```Java
package hw3;

public class dog extends animal{
	static int dog_sum=0;
	public dog()
	{
		
	}
	public void show()
	{
		super.sum++;
		dog_sum++;
		System.out.println("我是狗，是第"+dog_sum+"只狗");
		super.show();
	}
}

```

cat类

```Java
package hw3;

public class cat extends animal{
	static int cat_sum=0;
	public cat()
	{
		
	}
	public void show()
	{
		super.sum++;
		cat_sum++;
		System.out.println("我是猫，是第"+cat_sum+"只猫");
		super.show();
	}
}
```

pig类

```Java
package hw3;

public class pig extends animal{
	static int pig_sum=0;
	public pig()
	{
		
	}
	public void show()
	{
		super.sum++;
		pig_sum++;
		System.out.println("我是猪，是第"+pig_sum+"只猪");
		super.show();
	}

}

```

主类

```Java
package hw3;
import java.util.*;
public class main_animal {
	public static void show_interface(animal sc)
	{
		sc.show();
	}
	public static void main(String[] args)
	{
		Scanner s=new Scanner(System.in);
		animal an=new animal();
		while(true)
		{
			System.out.println("请输入添加的动物类型(dog,cat,pig)(输入exit退出程序)：");
			String m=s.nextLine();
			if(m.equals("exit"))
			{
				System.out.println("程序已退出");
				break;
			}
			switch(m)
			{
			case("dog"):
			{
				an=new dog();
				show_interface(an);
				break;
			}
			case("cat"):
			{
				an=new cat();
				show_interface(an);
				break;
			}
			case("pig"):
			{
				an=new pig();
				show_interface(an);
				break;
			}
			default:
			{
				System.out.println("非法字符,重新输入");
				break;
			}
			}
		}
	}
}

```

运行结果:

![QkAELR.png](https://s2.ax1x.com/2019/11/29/QkAELR.png)