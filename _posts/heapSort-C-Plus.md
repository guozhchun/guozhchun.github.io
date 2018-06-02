---
title: 堆排序（C++）
date: 2018-05-27 21:35:55
tags: [C/C++, 数据结构与算法, 排序]
categories: 数据结构与算法
---

```C++
#include <iostream>
using namespace std;

//  维护堆排序，采用下调的方式
void downAdjust(int k, int* a, int n) 
{
    int tem;
    if (2 * k + 1 >= n)  //  如果该结点是叶结点
        return;
    //  如果该结点只有左孩子
    if (2 * k + 2 >= n) 
    {
        //  如果该结点的值比左孩子的值小，则交换，然后继续下调操作
        //  否则停止下调操作
        if (a[k] < a[2 * k + 1]) 
        {
            tem = a[k];
            a[k] = a[2 * k + 1];
            a[2 * k + 1] = tem;
            downAdjust(2 * k + 1, a, n);
        }
        else
        {
            return;
        }
    } 
    else 
    {
        //  如果该结点值比左右孩子的值都大，则下调操作结束
        if (a[k] >= a[2 * k + 1] && a[k] >= a[2 * k + 2])
            return;
        //  如果该结点和左孩子都比右孩子小，则将该结点与右孩子交换，然后继续下调操作
        if (a[2 * k + 1] < a[2 * k + 2]) 
        {
            tem = a[k];
            a[k] = a[2 * k + 2];
            a[2 * k + 2] = tem;
            downAdjust(2 * k + 2, a, n);
        } 
        else 
        {
            tem = a[k];
            a[k] = a[2 * k + 1];
            a[2 * k + 1] = tem;
            downAdjust(2 * k + 1, a, n);
        }
    }
}

//  堆排序的函数
void heapsort(int* a, int n) 
{
    int tem;
    // 先把所有的值按照完全二叉树排好，然后运用下调方式维护堆，从而建立一个堆
    for (int i = n - 1; i >= 0; i--)
        downAdjust(i, a, n);
    //  将堆顶元素与最后一个元素交换，然后运用下调方式维护堆，直至排序完成
    for (int i = n - 1; i >= 0; i--) 
    {
        tem = a[0];
        a[0] = a[i];
        a[i] = tem;
        downAdjust(0, a, i);
    }
}

int main()
{
    int n;
    cout << "本程序采用堆排序对整数进行排序，以升序结果输出" << endl;
    cout << "请输入要排序的整数个数：";
    cin >> n;
    int* a = new int[n];
    cout << "请输入您要排序的数据:" << endl;
    for (int i = 0; i < n; i++)
        cin >> a[i];
    heapsort(a, n);
    cout << "排序后的结果是：" << endl;
    for (int i = 0; i < n; i++)
        cout << a[i] << " ";
    cout << endl;
    delete []a;
    return 0;
}

```

