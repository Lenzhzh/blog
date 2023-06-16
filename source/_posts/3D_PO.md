---
title: 三维偏序（CDQ）
date: 2023-06-16 16:38:50
tags: 
    - 算法
    - CDQ分治
    - 多维偏序
description: |
    二刷三维偏序模板题得到的一些感悟，有点收获。
---
# 三维偏序！

[**原题传送门**](https://www.luogu.com.cn/problem/P3810)

首先，三维偏序，顾名思义就是对一个含有3个维度的结构体，进行排序或比较的问题。在这道题中，我们要做的就是统计出权值为$k$的结构体的个数。

我们用node来存储，a,b,c分别表示三个维度，cnt表示同样的结构体的个数，ans表示权值小于等于这个结构体的元素个数。即

```cpp
struct node 
{
    int a,b,c;
    int ans,cnt;
}s1[LXB],s2[LXB];
```

那么，我们已经解决了读入的问题，该如何进行下一步操作呢？

## 排序！

对于多维偏序，我们基本的操作，就是一维一维地处理。如此题，我们先对结构体进行一维排序，按a,b,c优先度递减的原则进行从小到大排序。

```cpp
bool cmp(node x, node y){
    if(x.a == y.a){
        return x.b<y.b ? x.b<y.b : x.c<y.c;
    }
    return x.a<y.a;
}
```

排序之后，我们就能方便地处理相同的元素了。

```cpp
int tot=0,m=0;
F(n){
    tot++;
    if(s1[i] != s1[i+1]){//这里的写法简略的一下，现实中并不能这么做。
        s2[++m] = s1[i];
        s2[m].cnt = tot;
        tot = 0;
    }
}
```

处理完之后，我们就可以开始真正的分治环节了！

## CDQ分治！

对于分治算法，我的理解是，类似于归并排序的形式，先下分，后向上合并。这样对于每次处理，处理的都是一个相对有序的区间。

```cpp
void cdq(int l, int r)
{
    if(l == r) return ;
    int mid = (l + r) >> 1;
    cdq(l , mid);cdq(mid+1 , r);
    //向下分
}
```

之后我们正式开始里面的处理。想象一下，归并排序的方式就是，先人为让区间变得 **局部有序** 起来，至少使左区间的值全都小于右的值。那这里也是一样。

我们先对数据以b为关键词进行排序，然后计算c也符合条件的个数。

这样做的好处是，左边的a值一定小于右区间，且两边的b值却都单调递增，开始比较c值。搜到一个b较大就可以停止。至于个数，可以通过 **树状数组求前缀和** 达到。

而对于每个左区间的s2[i]，可以通过循环来处理。

```cpp
bool cmp(node x , node y)
{
    return x.b<y.b ? x.b<y.b : x.c<y.c;
}

void cdq(int l , int r)
{
    /*
    -------
    */
    sort(s2+l , s2+mid+1 , cmp2);
    sort(s2+mid+1 , s2+r+1 , cmp2);//排序后，确保左区间的a永远不大于右区间，且两边的b都单调递增。
    int j=l;
    F(mid){
        while(s2[i].b >= s2[j] and j <= mid)
        {
            add(s2[i].c , s2[i].cnt);//树状数组记录，从s2[j].c开始往上加到k，这样就可以防止由于s2[i].c较小导致的重复计算。
            j++；
        }
        s2[j].ans = query(s2[i].c);
    }
    memset(tr , 0 , sizeof(tr));
}
```

至此，大部分的过程已经解决了，统计 $f[i]$ 的时候，只要求出 $s2[i].ans+s2[i].cnt-1$ 就行了，因为ans记录的是三维均小的值，还需加上三维相同的值并减去自身

以下代码，应该很清楚了：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int LXB = 2e5 + 1145 ;

struct node
{
	int a,b,c,cnt,ans;
}s1[LXB],s2[maxn];


int n,m,k,mx,top,su[LXB];//su存储答案个数
int tr[LXB];//树状数组

bool cmp1(node x,node y)
{
	if(x.a==y.a)
	{
		if(x.b==y.b)return x.c<y.c;
		else return x.b<y.b;
	}
	else return x.a<y.a;
}//第一维排序

bool cmp2(node x,node y)
{
	if(x.b==y.b)
	return x.c<y.c;
	else return x.b<y.b;
}//第二维排序

int lowbit(int x)
{
	return x&(-x);
}

void add(int x,int y)
{
	while(x<=mx)
	{
		c[x]+=y;
		x+=lowbit(x);
	}
}//树状数组单点加

int query(int x)
{
	int sum=0;
	while(x)
	{
		sum+=c[x];
		x-=lowbit(x);
	}
	return sum;
}//求单点前缀和

void cdq(int l,int r)
{
	if(l==r)return;
	int mid=(l+r)>>1;
	cdq(l,mid);
	cdq(mid+1,r);//类似于归并排序
	sort(s2+l,s2+mid+1,cmp2);
	sort(s2+mid+1,s2+r+1,cmp2);//第二维为关键字排序
	int i,j=l;
	for(i=mid+1;i<=r;++i)
	{
		while(s2[i].b>=s2[j].b&&j<=mid)
		{
			add(s2[j].c,s2[j].cnt);//在s2[j]位置加上s2[j]的个数
			j++;
		}
		s2[i].ans+=query(s2[i].c);//保证树状数组里的数一定符合条件
	}//类似归并
	memset(tr , 0 , sizeof(tr));
}//cdq分治

int main()
{
	scanf("%d%d",&n,&k);
	mx=k;//树状数组的区间
	for(int i=1;i<=n;++i)
	{
		int a,b,c;
		scanf("%d%d%d",&a,&b,&c);
		s1[i].a=a;
		s1[i].b=b;
		s1[i].c=c;
	}//初始化输入
	sort(s1+1,s1+1+n,cmp1);//第一维为关键字排序
	for(int i=1;i<=n;++i)
	{
		top++;
		if(s1[i].a!=s1[i+1].a||s1[i].b!=s1[i+1].b||s1[i].c!=s1[i+1].c)
		{
			m++;
			s2[m].a=s1[i].a;
			s2[m].b=s1[i].b;
			s2[m].c=s1[i].c;
			s2[m].cnt=top;
			top=0;
		}
	}//第一维已有序,合并相同节点
	cdq(1,m);//cdq分治
	for(int i=1;i<=m;++i)su[s2[i].ans+s2[i].cnt-1]+=s2[i].cnt;
	for(int i=0;i<n;++i)printf("%d\n",su[i]);
	return 0;//为美好的世界献上return 0：
}
```








