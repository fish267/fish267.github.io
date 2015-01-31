---
author: Fish
layout: post
title: 快速排序递归与非递归实现
categories: C++
tags: algorithm
---
在 [多核计算与程序设计 ] (http://book.douban.com/subject/3624015/)一书上, 作者提到, 一般的商业应用中, 不会使用到所有的排序算法. 排序因数据多少, 导致效率差距非常大. 插入排序一般用于数据量较少的情况, 快速排序多少都可以; 归并排序用于数据比较多的情况, 而基数排序就用于数据非常多的情况( 30万以上 ).
<br>
![](http://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)
<br>
书上只提到了快速排序, 程序如下:
##递归快速排序
<!--more-->
{% highlight cpp %}
#include<iostream>
using namespace std;
/* Len函数, 获取数组元素个数 
*/
template<class T>
int Len(const T &Array){
    if(sizeof(Array) == 0)
        return 0;
    else
        return sizeof(Array) / sizeof(Array[0]);
}
/* display 函数
    @param Type *Array--任意类型数组的首地址
    @param int len--数组的元素个数
    @void -- 空数组输出None, 非空输出元素
*/
template< class T >
void display(const T &Array){
    if (Len(Array) == 0)
        cout << "None" << endl;
    else{
        for(int i = 0; i != Len(Array); i++)
            cout << *(Array + i) << " ";
        cout << endl;
    }
}
/* Swap函数
    @void -- 数组地址, 以及要交换的两个位置
*/
template< class Type >
void Swap(Type *pArray, int p, int q){
    Type temp = *(pArray + p);
    *(pArray + p) = *(pArray + q);
    *(pArray + q) = temp;
}
/* 分割函数
    @param Type *pArray--数组首地址
    @param int left--a[p:left] 小于
    @param int right--a[j:right] 大于
    @return int select--返回分界数据
*/
template< class Type >
int Split(Type *pArray, int startIndex, int endIndex){
    Type x = pArray[startIndex];
    while(startIndex < endIndex){
        while(pArray[startIndex] < x)
            startIndex++;
        while(pArray[endIndex] > x)
            endIndex--;
        if(pArray[startIndex] == pArray[endIndex])
            startIndex++;
        else if(startIndex < endIndex)
            Swap(pArray, startIndex, endIndex);
    }
    return endIndex;
}
/* 快速排序函数
   @param Type *pArray--数组首地址
   @param int start--起始位置
   @param int end --结尾数
*/
template< class Type >
void QuickSort(Type *pArray, int start, int end){
    if(start < end){
        int selectData = Split(pArray, start, end);         //选取分界数据
        QuickSort(pArray, start, selectData - 1);           //左边排序
        QuickSort(pArray, selectData + 1, end);             //右边排序
    }
}
/* if __name__ == '__main__' */
int main()
{
    int a[] = {
        1, 3, 2, 44, 55, 22,
        1, 3, 2, 44, 55, 22
    };
    QuickSort(a, 0, Len(a) - 1);
    display(a);
    return 0;
}
{% endhighlight %}
书上提到, 商业上不大使用递归, 对效率有很高的要求. 因此使用 C++ 标准库中的 stack 实现.
##非递归快速排序
{% highlight cpp %}
#include<iostream>
#include<stack>
using namespace std;
/* Len函数, 获取数组元素个数 
*/
template<class T>
int Len(const T &Array){
    if(sizeof(Array) == 0)
        return 0;
    else
        return sizeof(Array) / sizeof(Array[0]);
}
/* display 函数
    @param Type *Array--任意类型数组的首地址
    @param int len--数组的元素个数
    @void -- 空数组输出None, 非空输出元素
*/
template< class T >
void display(const T &Array){
    if (Len(Array) == 0)
        cout << "None" << endl;
    else{
        for(int i = 0; i != Len(Array); i++)
            cout << *(Array + i) << " ";
        cout << endl;
    }
}
/* Swap函数
    @void -- 数组地址, 以及要交换的两个位置
*/
template< class Type >
void Swap(Type *pArray, int p, int q){
    Type temp = *(pArray + p);
    *(pArray + p) = *(pArray + q);
    *(pArray + q) = temp;
}
/* 分割函数
    @param Type *pArray--数组首地址
    @param int left--a[p:left] 小于
    @param int right--a[j:right] 大于
    @return int select--返回分界数据
*/
template< class Type >
int Split(Type *pArray, int startIndex, int endIndex){
    Type x = pArray[startIndex];
    while(startIndex < endIndex){
        while(pArray[startIndex] < x)
            startIndex++;
        while(pArray[endIndex] > x)
            endIndex--;
        if(pArray[startIndex] == pArray[endIndex])
            startIndex++;
        else if(startIndex < endIndex)
            Swap(pArray, startIndex, endIndex);
    }
    return endIndex;
}
template< class T>
void QuickSort(T *pArray, int start, int end){
    stack< T > st;
    if (start < end){
        int mid = Split(pArray, start, end);
        if(start < mid - 1){
            st.push(start);
            st.push(mid - 1);
        }
        if(end > mid + 1){
            st.push(mid + 1);
            st.push(end);
        }
        while(!st.empty()){
            int  q = st.top();
            st.pop();
            int  p = st.top();
            st.pop();
            mid = Split(pArray, p, q);
            if(p < mid - 1){
                st.push(p);
                st.push(mid - 1);
            }
            if(q > mid + 1){
                st.push(mid + 1);
                st.push(q);
            }
        }
    }
}
int main(){
    float a[] = {
        12.2, 3, 5, 32, 1.3, 12.22, 12.21, 4,
    };
    display(a);
    QuickSort(a, 0, Len(a) - 1);
    display(a);
    return 0;
}
{% endhighlight %}

#END
比较来看, 使用堆栈不怎么省事... 因该是程序太简单了!


理论上, 递归算法的栈是程序自动生成的, 程序定义的局部变量都存放到栈中, 如果程序复杂, 栈就会很大, 在递归调用时, 需要先进行栈操作. 因此效率就低. 
