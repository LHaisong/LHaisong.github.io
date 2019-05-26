---
layout:     post
title:      各种排序算法大的比较
subtitle:   线程间通信
date:       2019-05-26
author:     LHaisong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - 排序算法
---

## 复杂度比较
![](https://i.imgur.com/mUhwDE1.png)

## 算法实现
- 冒泡排序
    /**
     *冒泡排序：
     *每次选择相邻的两个数，比较大小，将大数交换到后面
     **/
    void bubbleSort(int arr[],int len){
	for(int i=0;i<len;i++){
		for(int j=0;j<len-i-1;j++){
			if(arr[j]>arr[j+1]){
				swap(int a,int b);
			}
		}
	}
}
- 选择排序
    void SelectSort(int arr[],int len)
{
	int min=MIN_INT,index=0;
	for(int i=0;i<len;i++)
	{
		index=i;
		for(int j=i+1;j<len-1;j++)
		{
			if(arr[j]<arr[index])
			{
				index=j;
			}
		}
		if(index!=i)             //把找到的最小值放到无序区的最前面
		{
			int tmp=a[index];
			a[index]=a[i];
			a[i]=tmp;
		}
	}
}
- 插入排序
    void InsertSort(int arr[],int len)
{
	for(int i=1;i<len;i++)
	{
		if(a[i] < a[i-1]) //如果第i个元素比前面的元素小
	  {
	      int j = i-1;     //需要判断第i个元素与前面的多个元素的大小，换成j继续判断
          int x = a[i];    //将第i个元素复制为哨兵
	      while(j >= 0 && x < a[j]) //找哨兵的正确位置，比哨兵大的元素依次后移
	      {
            a[j+1] = a[j]; 
	        j--;
	      }
	      a[j+1] = x;  //把哨兵插入到正确的位置
	  }   
	}
}
- 快速排序
    int partition(vector<int> &vi, int l, int r) {
    int k = l, pivot = vi[r];
    for (int i = l; i < r; ++i){
        if (vi[i] <= pivot){
			swap(vi[i], vi[k++]);
		}
	}
    swap(vi[k], vi[r]);
    return k;
}
    void quicksort(vector<int> &vi, int l, int r) {
    if (l < r) {
        int pivot = partition(vi, l, r);
        quicksort(vi, l, pivot-1);
        quicksort(vi, pivot+1, r);
    }
}
- 归并排序
    void Merge(int a[], int low, int mid, int high)
{
	int n1 = mid-low+1;
	int n2 = high-mid;
	int *L = new int[n1+1];
	int *R = new int[n2+1];
	int i, j, k;
	
	for (i=0; i<n1; i++){
		L[i] = a[low+i];
	}
	for (j=0; j<n2; j++){
		R[j] = a[mid+j+1];
	}
	L[n1] = 10000000;
	R[n2] = 10000000;
 
	for (i=0, j=0, k=low; k<=high; k++)
	{
		if (L[i]<=R[j])
		{
			a[k] = L[i];
			i++;
		}else{
			a[k] = R[j];
			j++;
		}
	}
 
	delete []L;
	delete []R;
}
    void MergeSort(int a[], int low, int high)
{
	if (low<high)
	{
		int mid= (low+high)>>1;
		MergeSort(a, low, mid);
		MergeSort(a, mid+1, high);
		Merge(a, low, mid, high);
	}
}
