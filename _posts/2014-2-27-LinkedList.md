---
author: Fish
title: 链表的基本操作
layout: post
categories: C++
tags: algorithm
---

#单链表
单链表的操作, 包括初始化, 头插入, 尾巴插入, 查询删除节点, 删除链表等等操作.


链表反转没有完成, 程序应该使用类来弄的, 没有实现..CODE 如下:
<!--more-->
{% highlight cpp %}
#include <iostream>
using namespace std;

typedef struct Node_st{
    int data;
    Node_st *next;
}Node;
//only for the first node 
void initNode(Node *head, int n){
    head->data = n;
    head->next = NULL;
}
//在结尾添加, 需要遍历到最后一个
void addNode(Node *head, int n){
    Node *newNode = new Node;
    newNode->data = n;
    newNode->next = NULL;
    Node *cur = head;
    while(cur){
        if(cur->next == NULL){
            cur->next = newNode;
            return;
        }
        cur = cur->next;
    }
}
//insertFront
void insertFront(Node **head, int n){
    Node *newNode = new Node;
    newNode->data = n;
    newNode->next = *head;
    *head = newNode;
}
//search
Node *searchNode(Node *head, int n){
    Node *cur = head;
    while(cur){
        if(cur->data == n)
            return cur;
        cur = cur->next;
    }
    cout << "Can not find node " << n << " in the list " << endl;
}
//delete
bool deleteNode(Node **head, Node *ptrDel){
    Node *cur = *head;
    if(ptrDel == *head){
        *head = cur->next;
        delete ptrDel;
        return true;
    }
    while(cur){
        if(cur->next == ptrDel){
            cur->next = ptrDel->next;
            delete ptrDel;
            return true;
        }
        cur = cur->next;
    }
    return false;
}
//copy a list
void copyList(Node *node, Node **pNew){
    if(node != NULL){
        *pNew = new Node;
        (*pNew)->data = node->data;
        (*pNew)->next = NULL;
        copyList(node->next, &((*pNew)->next));
    }
}
//比较两个链表是否一样
bool compareList(Node *node1, Node *node2){
    bool flag;
    if(node1 == NULL && node2 == NULL)
        flag = true;
    else{
        if(node1 == NULL || node2 == NULL)
            flag = false;
        else if(node1->data != node2->data)
            flag = false;
        else
            compareList(node1->next, node2->next);
    }
}
//delete the linklist
void deleteList(Node **node){
   Node *tempNode;
   while(*node){
       tempNode = *node;
       *node = tempNode->next;
       delete tempNode;
   }
}
//display
void display(Node *head){
    Node *list = head;
    while(list){
        cout << list->data << " ";
        list = list->next;
    }
    cout << endl;
}
//test
//if __name__ == '__main__'
int main(){
    Node *newHead;
    Node *head = new Node;

    initNode(head, 111);
    display(head);
    
    addNode(head, 22);
    display(head);

    addNode(head, 333);
    display(head);

    addNode(head, 555);
    display(head);

    addNode(head, 777);
    display(head);

    insertFront(&head, 1024);
    display(head);

    int numDel = 22;
    Node *searcNum = searchNode(head, numDel);
    if(deleteNode(&head, searcNum))
        cout << "Node " << numDel << " deleted !" << endl;
    display(head);

    cout << "the list is copied: " << endl;
    copyList(head, &newHead);
    display(newHead);

    cout << "comparing the two lists" << endl;
    cout << "Are the two lists the same ? " << endl;
    if(compareList(head, newHead))
        cout << "yes, they are the same " << endl;
    else
        cout << "No, not the same." << endl;

    numDel = 333;
    searcNum = searchNode(newHead, numDel);
    if(deleteNode(&newHead, searcNum)){
        cout << " node " << numDel << "is deleted in newHead" << endl;
        cout << "the newHead after deleted is " << endl;
        display(newHead);
    }
    cout << "comparing the two lists" << endl;
    cout << "Are the two lists the same ? " << endl;
    if(compareList(head, newHead))
        cout << "yes, they are the same " << endl;
    else
        cout << "No, not the same." << endl;

    cout << endl;
    deleteList(&newHead);
    display(head);
    deleteList(&head);
    return 0;
}
{% endhighlight %}
<b>上面测试运行结果如下:</b>
<pre>
111 
111 22 
111 22 333 
111 22 333 555 
111 22 333 555 777 
1024 111 22 333 555 777 
Node 22 deleted !
1024 111 333 555 777 
the list is copied: 
1024 111 333 555 777 
comparing the two lists
Are the two lists the same ? 
No, not the same.
 node 333is deleted in newHead
the newHead after deleted is 
1024 111 555 777 
comparing the two lists
Are the two lists the same ? 
yes, they are the same 

1024 111 333 555 777 
[Finished in 0.2s]
</pre>
