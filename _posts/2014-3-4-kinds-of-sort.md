---
author: Fish
layout: post
title: 各大排序总结
categories: C++
tags: algorithm
---
昨天为了准备一个公司的实习招聘笔试, 看了一晚上的面试宝典, 书上的排序算法挺多, 但是有些繁琐, 自己重新总结了一下.


具体的排序算法介绍, 随后补充, 今天先贴代码, 注释和解释有空了添加进来:
<!--more-->
{% highlight cpp %}
#include <iostream>
#include <cstdlib>
using namespace std;
// 初始化数组
void initArray(int array[], int len, int max){
    srand(time(NULL));
    for(int i = 0; i < len; i++)
        array[i] = rand() % max; 
}
//display
void display(int *array, int len){
    for(int i = 0; i < len; i++)
        cout << array[i] << " ";
    cout << endl;
}
//二分法查找
int binarySearch(int array[], int len, int searchData){
    int start = 0;
    int end = len - 1;
    while(start < end){
        int mid = (start + end ) / 2;
        if(array[mid] == searchData)
            return mid;
        else if(array[mid] > searchData)
            start = mid + 1;
        else
            end = mid - 1;
    }
    return -1;
}
//交换数组中两个元素
void swap(int *array, int pos1, int pos2){
    int temp = array[pos1];
    array[pos1] = array[pos2];
    array[pos2] = temp;
}
/************************************/
void BubbleSort(int array[], int len){
    for(int i = 0; i < len - 1; i++)
        for(int j = i + 1; j < len; j ++){
            if(array[i] > array[j])
                swap(array, i, j);
        }
}
/************************************/
void SelectSort(int *array, int len){
    for(int i = 0; i < len - 1; i++){
        for(int j = i + 1; j < len; j++){
            int min = array[i];
            if(min > array[j])
                swap(array, i, j);
        }
    }
}
/************************************/
void InsertSort(int *array, int len){
    for(int j  = 1; j  < len; j++){
        for(; j >= 0 && array[j - 1] > array[j]; j--)
            swap(array, j , j - 1);
    }
}
/************************************/
int Split(int *array, int start, int end){
    int temp = array[end];
    while(start < end){
        while(array[start] < temp)
            start++;
        while(array[end] > temp)
            end--;
        if(array[start] == array[end])
            start++;
        else if(start <  end)
            swap(array, start, end);
    }
    return end;
}
void QuickSort(int *array, int start,  int len){
    int end = len;
    if(start < end){
        int temp = Split(array, start, end);
        QuickSort(array, start, temp - 1);
        QuickSort(array, temp + 1, end);
    }
}
/************************************/
void ShellSort(int *array, int len){
    for(int gap = len / 2; gap > 0; gap /= 2)
        for(int i = gap; i< len; i++)
            for(int j = i - gap; j >=0 && array[j] > array[j + gap]; j -= gap)
                swap(array, j, j + gap);
}
/************************************
fake code:
go on mergesort:
    if n < 2:
        return
    else:
        sort the left half
        sort the right half
        merge two above
*************************************/
void MergeArray(int *array, int start, int mid, int end){
    int *p = new int[end - start + 1];
    int i = start, j = mid + 1;
    int k = 0;
    while(i <= mid && j <= end)
        p[k++] = array[i] <= array[j]? array[i++] : array[j++];
    while(i <= mid)
        p[k++] = array[i++]; 
    while(j <= end)
        p[k++] = array[j++];
    for(int t = 0; t < k; t++)
        array[start + t] = p[t];
    delete p;
}
void MergeSort(int *array, int start, int end){
    if(start < end){
        int mid = (start + end) / 2;
        MergeSort(array, start, mid);
        MergeSort(array, mid + 1, end);
        MergeArray(array, start, mid, end);
    }
    
}
/************************************/
//还没看懂,稍后补充
void BuildMaxHeap(int p[], int size);
void DeleteMax(int p[], int size);
void PercolateDown(int array[],int hole, int size);
void HeapSort(int p[], int size)
{
    BuildMaxHeap(p, size);
    for (int i=size; i>=1; i--)
    {
        DeleteMax(p, i);
    }
}


void BuildMaxHeap(int p[], int size)
{
    //由于所有树叶无需进行下滤（没有孩子）， 所以只对0 - size/2的结点进行下滤即可。
    for (int i=size/2-1; i>=0; i--)
        PercolateDown(p, i, size);
}

void DeleteMax(int p[], int size)
{
    int tmp;
    tmp = p[0];
    p[0] = p[size-1];
    p[size-1] = tmp;
    PercolateDown(p, 0, --size);   
}

void PercolateDown(int array[],int hole, int size)
{
    int tmp = array[hole];
    int child=0;
    for (; (hole*2+1)<=(size-1); hole=child)
    {
        child = hole*2 + 1;
        if (child<size-1 && (array[child+1] > array[child]))
            child++;
        if (array[child]>tmp)
            array[hole] = array[child];
        else
            break;
    }
    array[hole] = tmp;
}
/*************************************************************/
int main(){
    const int SIZE = 15;
    int *array = new int[SIZE];
    
    cout << "BubbleSort: " << endl;
    initArray(array,SIZE, 100);
    display(array, SIZE);
    BubbleSort(array, SIZE);
    display(array, SIZE);
    cout << "************************" << endl;

    cout << "SelectSort: " << endl;
    initArray(array,SIZE, 100);
    display(array, SIZE);
    SelectSort(array, SIZE);
    display(array, SIZE);
    cout << "************************" << endl;

    cout << "InsertSort: " << endl;
    initArray(array,SIZE, 100);
    display(array, SIZE);
    InsertSort(array, SIZE);
    display(array, SIZE);
    cout << "***********************" << endl;

    cout << "QuickSort: " << endl;
    initArray(array,SIZE, 100);
    display(array, SIZE);
    QuickSort(array, 0, 14);
    display(array, SIZE);
    cout << "***********************" << endl;

    cout << "ShellSort: " << endl;
    initArray(array,SIZE, 100);
    display(array, SIZE);
    ShellSort(array, SIZE);
    display(array, SIZE);
    cout << "***********************" << endl;

    cout << "MergeSort: " << endl;
    initArray(array,SIZE, 100);
    display(array, SIZE);
    MergeSort(array, 0, 14);
    display(array, SIZE);
    cout << "***********************" << endl;

    cout << "HeapSort: " << endl;
    initArray(array,SIZE, 100);
    display(array, SIZE);
    HeapSort(array, SIZE);
    display(array, SIZE);
    cout << "***********************" << endl;


    return 0;
}
{% endhighlight %}
运行结果如下:
    
    
    BubbleSort: 
    15 82 83 24 45 65 49 12 29 98 66 89 95 39 33 
    12 15 24 29 33 39 45 49 65 66 82 83 89 95 98 
    ************************
    SelectSort: 
    15 82 83 24 45 65 49 12 29 98 66 89 95 39 33 
    12 15 24 29 33 39 45 49 65 66 82 83 89 95 98 
    ************************
    InsertSort: 
    15 82 83 24 45 65 49 12 29 98 66 89 95 39 33 
    12 15 24 29 33 39 45 49 65 66 82 83 89 95 98 
    ***********************
    QuickSort: 
    15 82 83 24 45 65 49 12 29 98 66 89 95 39 33 
    12 15 24 29 33 39 45 49 65 66 82 83 89 95 98 
    ***********************
    ShellSort: 
    15 82 83 24 45 65 49 12 29 98 66 89 95 39 33 
    12 15 24 29 33 39 45 49 65 66 82 83 89 95 98 
    ***********************
    MergeSort: 
    15 82 83 24 45 65 49 12 29 98 66 89 95 39 33 
    12 15 24 29 33 39 45 49 65 66 82 83 89 95 98 
    ***********************
    HeapSort: 
    15 82 83 24 45 65 49 12 29 98 66 89 95 39 33 
    12 15 24 29 33 39 45 49 65 66 82 83 89 95 98 
    ***********************
