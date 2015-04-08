---
author: Fish
layout: post
title: Java学习笔记 
categories: ali 
tags: java 
---

工作需求，猛看Java，学习完了马士兵老师的第一阶段视频，做一下记录和总结。

##Hello world

HelloWorld.java

{%highlight java %}
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello world !");
    }
}
{%endhighlight%}

##变量

    byte int short long float double char boolean

##数组

    int a[];
    int [] a = new int[100];
    int [] b = new int[] {1, 2, 3};

#类

FaceObject.java
{%highlight java %}
interface dot {
    void fishing();
}
public class FaceObject {
    public static void main(String[] args) {
        Fish f = new Fish();
        f.fishing();
    }
}

class Fish implents dot{

    private int amount;

    void fishing() {
        System.out.println("Fsh is fishing");
    }
    
    int getAmount() {
        return amount;
    }

    Fish(int amount) {
        this.amount = amount;
    }

    Fish() {
        this.amount = 0;
    }
}
{%endhighlight%}

#END
