---
title: test
date: 2019-10-10 20:56:07
tags: [test]
---
<!--more-->
```
#include<bits/stdc++.h>
using namespace std;
int days[13] = {0,31,28,31,30,31,30,31,31,30,31,30,31};
bool isLeap(int y)
{
    if(y%400 == 0) return true;
    if(y%4 == 0 && y%100) return true;
    return false;
}
int main()
{
    int y,m,d;
    while(cin>>y>>m>>d)
    {
        bool flag = isLeap(y);
        if(m <= 0 || m > 12 || d <= 0)
        {
            cout<<"0"<<endl;
            continue;
        }
        int dd = days[m];
        if(m == 2 && flag) 
			dd++;
        if(d <= dd)
        {
            cout<<"1"<<endl;
            continue;
        }
        cout<<"0"<<endl;
    }
    return 0;
}
```