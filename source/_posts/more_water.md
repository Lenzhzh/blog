---
title: 更多的水源！摩多摩多哟
date: 2023-07-11 13:52:24
tags: 
    - 算法
    - 线段树
    - 数据结构
description: |
    灵感来源于[降雨量](https://www.luogu.com.cn/problem/P2471)，比较好玩的一题，考察了对多种情况的分析和线段树的应用，实际上手大概2小时能打完。
    ——更进一步！
---
# 更进一步的理解！

在刷题的时候又刷到了这道题，怎么说呢，像是在利用线段树为工具，而并非对算法本身的理解。其实我们对于线段树的理解本身就应该是这样的。

对于线段树，只需要把他视为一个能以$O(logn)$的复杂度直接输出区间内的某个值，比如说最大最小值，区间和，区间极差等等。

所以，_做类似的题目，直接跳过线段树设计，先对基本算法进行分析_ 会好得多。这也是为什么线段树是一个**数据结构**的原因！

# 更多降雨！

对于[降雨量](https://www.luogu.com.cn/problem/P2471)这题，如果以上面的思路来说，估计还是比较简单的，只需要用线段树维护一个区间的最大值即可。

但如果用线段树维护 $[l,r]$ 年份之间的最大降雨量，我们会发现降雨量的范围比较离谱，于是我们自然想到了———————

## 读入方式！

嘿嘿，想不到吧，不是离散化！由于输入已经保证年份单调递增，所以可以直接按顺序读入 $yr_{i},co_{i}$ ，作为年份和降雨量，然后用i（即年份编号）来带入线段树。

所以不难看出，线段树和离散化还是很契合的！只是这道题恰好不需要而已。由于时间很快，所以即使是离散化，这题应该也是可以通过的，只是代码略微复杂而已！

## 算法设计！

都说先不考虑线段树，于是我们直接考虑如何得出答案。这题有三种情况，而排除前两种，剩下的就是最后一种情况 ~~这不废话吗~~ ，所以我们从对错两种开始考虑 ~~（主要是maybe太多了，当然看了题解之后发现其实也差不多）~~ 

我们将每次读到的两个年份记为x,y，第一个不超过x的年份编号记为st，y的记为ed，用线段树得到的区间最大值记为tmp，下面是几种情况：

> 错误的情况
> 1. x>=y ，预知未来，没什么好说的。
> 2. tmp>(co[x]或co[y]) ， 中间报表，没什么好说的。
> 3. co[x]<co[y],且x==st,y==ed ，当xy全部已知降雨量时，后面比前面大。与原题意一样，但在现实中其实也说的通！切记！

> 正确的条件（全部满足才算正确）
> 1. 不错误 ~~废话~~
> 2. 中间全部连上(ed - st == yr[ed] - yr[st])，前者代表存在年份编号（下表i）的年份数量，后者代表年份数量，如果二者相等才是全覆盖
> 3. 左右端点已知(yr[st] == x and yr[ed] == y)

剩下就是maybe情况了！

## 具体实现！

我们首先得找出st，而st是年份编号，用lower_bound获得 yr 中第一个不大于 x 的下标，y同理。然后全都往里缩一个单位，获得要处理的操作区间，这个操作区间是 $[x,y]$ 之内的，最长的两端点均已知的区间。

接着用fl和fr来分别表示，x，y的降雨量是否已知。

```cpp
int st = lower_bound(yr+1 , yr+n+1 , x)-yr;
int ed = lower_bound(yr+1 , yr+n+1 , y)-yr;
bool fl = (x==yr[st]) , fr = (yr[ed]==y);
if(!fl)st--;//如果x不被包括，那么st就是x后最近的已知降雨年份，如果区间会被用到的话，直接使用st即可，否则是操作区间左端点是st+1
int tmp = 0;
if(st+1 <= ed-1)tmp = out(st+1 , ed-1 , 1);//判断区间是否存在，如果存在求最大值
```

接下来，用tmp和co[x]，co[y]比较，正确条件判断等，不多赘述

至于线段树的应用可以参考[前面的博客](https://son.florance.eu.org/2023/06/27/max_in_ltor/)！

以下代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define str string
#define db    double
#define DG(n)    cout<<n<<endl;
#define re(n)    n=read();
#define F(n)    for(int i=1;i<=n;i++)
#define nel    (rt<<1)
#define ner    (rt<<1|1)
#define L (tr[rt].l)
#define R (tr[rt].r)
#define lson    l,r,nel
#define rson    l,r,ner
#define trm    ((tr[rt].l+tr[rt].r)>>1)
#define trf    ((tr[rt].r-tr[rt].l))

const int LXB=2e5+10;
const int lxb=3e3;

int n,m,k,l,t;
string s;
char p;

int yr[LXB],co[LXB];

struct tree{
    int l,r,x;
}tr[LXB];


void pup(int rt){tr[rt].x = max(tr[nel].x , tr[ner].x);}


void build(int l , int r , int rt)
{
    tr[rt].l = l , tr[rt].r = r;
    if (l==r) {tr[rt].x = co[l] ; return ;}
    build(l , trm , nel);build(trm+1 , r , ner);
    pup(rt);
}


int out(int l , int r , int rt)
{
    int ans = 0;
    if(l <= tr[rt].l and tr[rt].r <= r){
        return tr[rt].x;
    }
    if(l<=trm) ans = max(ans , out(l , r , nel));
    if(r>trm) ans = max(ans , out(l , r , ner));
    return ans;
}//求区间最大值的线段树，可以参考前面的博客


signed main(){
    ios::sync_with_stdio(0);cin.tie(0);
    int T=1;
    
    while(T--){
        cin>>n;
        F(n){
            cin>>yr[i]>>co[i];
        }
        build(1,n,1);
        cin>>m;
        int x,y;
        F(m){
            cin>>x>>y;
            if(x >= y){ cout<<"false"<<endl; continue;}
            int st = lower_bound(yr+1 , yr+n+1 , x)-yr;
            int ed = lower_bound(yr+1 , yr+n+1 , y)-yr;
            bool fl = (x==yr[st]) , fr = (yr[ed]==y);
            if(!fl)st--;
            int tmp = 0;
            if(st+1 <= ed-1)tmp = out(st+1 , ed-1 , 1);
            if((tmp>=co[ed] && fr)||(tmp>=co[st] && fl)||(co[st]<co[ed] && fl && fr))
                {cout<<"false"<<endl;continue;}//错误条件判断。从左到右分别：是中间大于右边，中间大于左边，右边大于左边
            if((ed-st == yr[ed]-yr[st]) and fl and fr)
                {cout<<"true"<<endl;continue;}//正确条件判断，全部满足才行，顺序：未已知，左端点未知，右端点未知
            cout<<"maybe"<<endl;
        }
    }
    return 0;//为，美，0
}

```





