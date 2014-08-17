---
author: fish
layout: post
title: 一个日期类-cpp
categories:
- Cpp 
tags: algorithm
---
设计一个日期类Date, 包括年月日，要求实现日期的基本运算， 如某个日期加上天数， 两个日期的间隔天数， 等。

在程序中重载下运算符，这样比较直观：

- Date operator+(int days)
- Date operator-(int days)
- int operator-(Date &d)

私有成员函数:

- leap(int)
- dton(Date &) // 指定日期到0-0-0开始的天数
- ntod(int)    // 将天数转换成日期

<!--more-->
程序如下：
{% highlight cpp %}

#include<iostream>
#include<ctime>
using namespace std;

int day_tab[2][12] = {{31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},
                        {31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}};

class Date{
    private:
        int year, month, day;
        int leap(int);
        int dton(Date &);
        Date ntod(int);
    public:
        Date();
        Date(int y, int m, int d){ year = y; month = m; day = d;}
        void setday(int d){ day = d;}
        void setyear(int y){year = y;}
        void setmonth(int m){month = m;}
        int getday(){return day;}
        int getyear(){return year;}
        int getmonth(){return month;}

        Date operator+(int days){
            static Date date;
            int number = dton(*this) + days;
            return ntod(number);
        }
        Date operator-(int days){
            static Date date;
            int number = dton(*this) - days;
            return ntod(number);
        }
        int  operator-(Date &d){
            return dton(*this) - dton(d);
        }
        void disp(){
            cout << year << "-" << month << "-" << day << endl;
        }

};
Date::Date(){
    time_t tt;
    time(&tt);
    struct tm *p;
    p = localtime(&tt);
    year = p->tm_year + 1900;
    month = p->tm_mon + 1;
    day = p->tm_mday;
}
int Date::leap(int year){
    if(year % 4 == 0 && year % 100 != 0 || year % 400 == 0)
        return 1;
    else
        return 0;
}

int Date::dton(Date &d){
    int ret = 0;
    int year = d.getyear();
    for(int i = 0; i < year; i++){
        if(leap(i))
            ret += 366;
        else
            ret += 365;
    }
    for(int i = 0; i < d.getmonth() - 1; i++){
        if(leap(year))
            ret += day_tab[1][i];
        else
            ret += day_tab[0][i];
    }
    return ret + d.getday();
}
// 将天数转换成日期
Date Date::ntod(int days){
    int rest = days;
    int y = 0, m = 1;
    while(true){
        if(leap(y)){
            if(rest <= 366)
                break;
            else
                rest -= 366;
        }
        else{
            if(rest <= 365)
                break;
            else
                rest -= 365;
        }
        y++;
    }

    while(true){
        if(leap(y)){
            if(rest > day_tab[1][m - 1])

                rest -= day_tab[1][m - 1];
            else
                break;
        }
        else{
            if(rest > day_tab[0][m - 1])
                rest -= day_tab[0][m - 1];
            else
                break;
        }
        m++;
    }
    return Date(y, m, rest);
}
int main(){

    Date d;
    Date then(2011, 2, 26);
    cout << d - then << endl;
    return 0;
}
{% endhighlight %}

#END
