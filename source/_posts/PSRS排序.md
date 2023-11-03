---
title: 使用C++实现PSRS排序的并行计算
date: 2021.11.8
author: Aik
img: /source/images/xxx.jpg
top: false
hide: false
cover: false
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: 使用PSRS排序算法进行并行计算
categories: 并行计算
tags:
  - C++
  - PSRS
---

# 使用C++实现PSRS排序的并行计算

随机生成1w个数并进行排序

```c++
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<omp.h>

#include <cmath>
#include <ctime>
#define num_threads 10

int *L,*R;

//Merge函数合并两个子数组形成单一的已排好序的字数组
//并代替当前的子数组A[p..r] 
//void Merge(int *a, int p, int q, int r)
//{
//    int i,j,k;
//    int n1 = q - p + 1;
//    int n2 = r - q;
//    L = (int*)malloc((n1+1)*sizeof(int));
//    R = (int*)malloc((n2+1)*sizeof(int));
//    for(i=0; i<n1; i++)
//        L[i] = a[p+i];
//    L[i] = 65536;
//    for(j=0; j<n2; j++)
//        R[j] = a[q+j+1];
//    R[j] = 65536;
//    i=0,j=0;
//    for(k=p; k<=r; k++){
//        if(L[i]<=R[j]){
//            a[k] = L[i];
//            i++;
//        }
//        else{
//            a[k] = R[j];
//            j++;
//        }
//    }
//}
template<typename  T>
void Merge(T arr[], int l, int mid, int r){

    //* VS不支持动态长度数组, 即不能使用 T aux[r-l+1]的方式申请aux的空间
    //* 使用VS的同学, 请使用new的方式申请aux空间
    //* 使用new申请空间, 不要忘了在__merge函数的最后, delete掉申请的空间:)
    T aux[r-l+1];
    //T *aux = new T[r-l+1];

    for( int i = l ; i <= r; i ++ )
        aux[i-l] = arr[i];

    // 初始化，i指向左半部分的起始索引位置l；j指向右半部分起始索引位置mid+1
    int i = l, j = mid+1;
    for( int k = l ; k <= r; k ++ ){

        if( i > mid ){  // 如果左半部分元素已经全部处理完毕
            arr[k] = aux[j-l]; j ++;
        }
        else if( j > r ){  // 如果右半部分元素已经全部处理完毕
            arr[k] = aux[i-l]; i ++;
        }
        else if( aux[i-l] < aux[j-l] ) {  // 左半部分所指元素 < 右半部分所指元素
            arr[k] = aux[i-l]; i ++;
        }
        else{  // 左半部分所指元素 >= 右半部分所指元素
            arr[k] = aux[j-l]; j ++;
        }
    }

    //delete[] aux;
} 
//归并排序
void MergeSort(int *a, int p, int r)
{
    if(p<r){
        int q = (p+r)/2;
        MergeSort(a,p,q);
        MergeSort(a,q+1,r);
        Merge(a,p,q,r);
    }
} 

void PSRS(int *array, int n)
{
    int id;
    int i=0;
    int count[num_threads][num_threads] = { 0 };    //每个处理器每段的个数
    int base = n / num_threads;     //划分的每段段数
    int p[num_threads*num_threads]; //正则采样数为p
    int pivot[num_threads-1];       //主元
    int pivot_array[num_threads][num_threads][5000]={0};  //处理器数组空间

    omp_set_num_threads(num_threads);
#pragma omp parallel shared(base,array,n,i,p,pivot,count) private(id)
    {
        id = omp_get_thread_num();
        //每个处理器对所在的段进行局部串行归并排序
        MergeSort(array,id*base,(id+1)*base-1);

#pragma omp critical
        //每个处理器选出P个样本，进行正则采样
        for(int k=0; k<num_threads; k++)
            p[i++] = array[(id-1)*base+(k+1)*base/(num_threads+1)];
//设置路障，同步队列中的所有线程
#pragma omp barrier
//主线程对采样的p个样本进行排序
#pragma omp master
        {
            MergeSort(p,0,i-1);
            //选出num_threads-1个主元
            for(int m=0; m<num_threads-1; m++)
                pivot[m] = p[(m+1)*num_threads];
        }
#pragma omp barrier
        //根据主元对每一个cpu数据段进行划分
        for(int k=0; k<base; k++)
        {
            for(int m=0; m<num_threads; m++)
            {
                if(array[id*base+k] < pivot[m])
                {
                    pivot_array[id][m][count[id][m]++] = array[id*base+k];
                    break;
                }
                else if(m == num_threads-1) //最后一段
                {
                    pivot_array[id][m][count[id][m]++] = array[id*base+k];
                }
            }
        }
    }
//向各个线程发送数据，各个线程自己排序
#pragma omp parallel shared(pivot_array,count)
    {
        int id=omp_get_thread_num();
        for(int k=0; k<num_threads; k++)
        {
            if(k!=id)
            {
                memcpy(pivot_array[id][id]+count[id][id],pivot_array[k][id],sizeof(int)*count[k][id]);
                count[id][id] += count[k][id];
            }
        }
        MergeSort(pivot_array[id][id],0,count[id][id]-1);
    }
    i = 0;
    printf("result:\n");
    for(int k=0; k<num_threads; k++)
    {
        for(int m=0; m<count[k][k]; m++)
        {
            printf("%d ",pivot_array[k][k][m]);
        }
        printf("\n");
    }
}
int* GetRandom(){
    srand((unsigned int)time(0));//初始化种子为随机值
    int i = 0;
    static int random[10000];
    for(;i < 10000;++i){
        int num = rand() % 100000 + 1;//产生一个1-50之间的数
        //printf("%d:",i);
        //printf("%d \n",num);
        random[i] = num;
    }
    printf("\n");
    return random;
}
int main()
{
    //int array[36] = {16,2,17,24,33,28,30,1,0,27,9,25,34,23,19,18,11,7,21,13,8,35,12,29,6,3,4,14,22,15,32,10,26,31,20,5};
    int* array = new int[10000];
    array = GetRandom();
    double begin,end,time;
    begin = omp_get_wtime();
    PSRS(array, 10000);
/*  MergeSort(list,0,35);
    for(int i=0; i<36; i++)
    {
        printf("%d ",list[i]);
    }*/
    end = omp_get_wtime();
    time = end - begin;
    printf("The running time is %lfs\n",time);
    return 0;
}
```



参考网址：

https://coding.imooc.com/learn/questiondetail/102979.html

https://github.com/liuyubobobo/Play-with-Algorithms/blob/master/03-Sorting-Advance/Course%20Code%20(C%2B%2B)/02-Merge-Sort/main.cpp
