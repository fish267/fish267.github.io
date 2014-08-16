---
author: fish
layout: post
title: 二叉树-cpp 
categories:
- Cpp 
tags: algorithm
---

建立一个二叉树，然后中序遍历一下，存储过程留意写， 代码实现如下：
<!--more-->

{% highlight cpp %}
#include<iostream>
using namespace std;

struct tree{
    int data;
    tree *right, *left;
};

class Btree{
    private:
        tree *root;
    public:
        Btree(){root = NULL;}
        void create_btree(int);
        void display(){ inorder(root); cout << endl;}
        void inorder(tree *);
};

void Btree::create_btree(int x){
    tree *newnode = new tree;
    newnode->data = x;
    newnode->right = newnode->left = NULL;
    if(root == NULL)
        root = newnode;
    else{
        tree *back;
        tree *current = root;
        while(current != NULL){
            back = current;
            if(current->data > x)
                current = current->left;
    
            else
                current = current->right;
        }
        if(back->data > x)
            back->left = newnode;
        else
            back->right = newnode;
    }
}
void Btree::inorder(tree *tmp){
    if(tmp != NULL){
        inorder(tmp->left);
        cout << tmp->data << " ";
        inorder(tmp->right);
    }
}
int main(){
    Btree a;
    for(int i = 0; i < 10; i++)
        a.create_btree(i);
    a.display();
    return 0;
}
{% endhighlight %}

#END
