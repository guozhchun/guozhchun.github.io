---
title: 快排（C++）
date: 2018-05-27 21:16:31
tags: [C/C++, 数据结构与算法, 排序]
categories: 数据结构与算法
---

```c++
#include <iostream>
using namespace std;

void swap(int& a, int& b)
{
    int temp;
    temp = a;
    a = b;
    b = temp;
}

void quick_sort(int* a, int start, int end)
{
    if (start >= end)
        return;
    int key = a[(start + end) / 2];
    int i = start, j = end;
    while (i <= j)  // the '=' can remove
    {
        // the '<' can not change to '<=', or it will wrong
        // for example, the array is 3, 3, 3, 3. In this way, the key is 3
        // if the condition is a[i] <= key, then i will increase to 4 which is out of the range of array
        while (a[i] < key)
            i++;
        // the '>' can not change to '>=', or it will wrong
        // for example, the array is 3, 3, 3, 3. In this way, the key is 3
        // if the condition is a[j] >= key, then i will decrease to -1 which is out of the range of array
        while (a[j] > key)
            j--;
        // must have this if condition, or it will wrong
        // for example, the array is 3, 5, 7, 1, 8, 6, 4, 2
        // if don't have the if condition, then the 5 will come to the left one and j will be -1
        // after that, the 5 will be in left one always, which is wrong obviously
        // must have "=", considerate the array 2, 3. if not "=", it will loop and can't stop
        if (i <= j)  
        {
            swap(a[i], a[j]);
            i++;
            j--;
        }
    }
    quick_sort(a, start, j);
    quick_sort(a, i, end);
}

int main()
{
    int n;
    cout << "please input the number count you want to sort: ";
    cin >> n;
    int* a = new int[n];
    cout << "please input the number, seperate with space" << endl;
    for (int i = 0; i < n; i++)
        cin >> a[i];
    quick_sort(a, 0, n - 1);
    cout << "the sort result is: ";
    for (int i = 0; i < n; i++)
        cout << a[i] << " ";
    cout << endl;
    delete []a;
    system("pause");
    return 0;
}


```

