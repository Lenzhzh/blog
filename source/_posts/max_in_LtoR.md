---
title: 防止土地沙漠化——线段树求区间最大值！
date: 2023-06-27 20:38:50
tags: 
    - 算法
    - 线段树
    - 扫描线
description: |
    灵感来源于一道题，叫【窗口的星星】，花了几节课想了一下这种题的方法，也算是第一次独立利用线段树解题的...
    ————所以这就是你以前抄题解的理由吗！
---
# 线段树求区间最大值！

线段树求区间最大值其实来源于一个《非常简单》的问题——

```
有一个长度为n数组，并给定两种操作：

1 l r x 表示在[l,r]的区间内每个数加上x;

2 l r 表示求[l,r]中的最小值;
```

于是，线段树求区间最大值横空出世！

## 基本分析

我们都知道，这种题在数据正常的情况下，是非常简单的。由于区间最大值查询的复杂度是 $O(n)$ ，而区间加减的复杂度也不高，所以《非常简单》，然而涉及到线段树，这道题就会有《与之适配》的数据范围了。

所以，我们要做的，就是建一棵树~~来防止土地沙漠化~~，由于防止土地沙漠化的复杂度是 $O(log(n))$ ，所以可以处理大部分情况。

但这只是原始线段树，我们要求一段的最大值，如果不对线段树进行根本上的改装，复杂度是要上天的。所以...

**我们对原始线段树进行改装，将存储数据变成一段区间的最大值就好了！**

我们都知道，对于一段区间的修改结束后，**这段区间的最大值就是原本最大值加上修改值**，所以这下修改就变得异常简单了。

```cpp
void pup(int rt){tr[rt].x = max(tr[ner].x , tr[nel].x);}

void up(int l,int r,int rt,int x)
{
    if(l <= tr[rt].l and tr[rt].r <= r){
        tr[rt].x += x ; add[rt]+=x;
        return ;
    }
    pdown(rt);
    int mid = trm;
    if(l<=mid)up(lson,x);
    if (r>mid)up(rson,x);
    pup(rt);
}

```

那么，再套上原本的板子，我们的线段树主题就圆满地实现了~

```cpp
void pup(int rt){tr[rt].x = max(tr[ner].x , tr[nel].x);}

void build(int l,int r,int rt)
{
    tr[rt].r = r , tr[rt].l = l;
    if(l==r)return;
    int mid = trm;
    build(l,mid,nel);
    build(mid+1,r,ner);
}

void pdown(int rt)
{
    add[nel] += add[rt];
    add[ner] += add[rt];
    tr[nel].x += add[rt];
    tr[ner].x += add[rt];
    add[rt] = 0;
}


void up(int l,int r,int rt,int x)
{
    if(l <= tr[rt].l and tr[rt].r <= r){
        tr[rt].x += x ; add[rt]+=x ; 
        return ;
    }
    pdown(rt);
    int mid = trm;
    if(l<=mid)up(lson,x);
    if (r>mid)up(rson,x);
    pup(rt);
}
```

# 回到正题，窗口的星星！

看到这道题，一开始我其实没有什么思路，但略加思索~查看题解~后，便明白了这道题的精髓。

我们将每个星星扩张成一个与窗口一样大，且以星星为左下角的矩形，再把窗口缩小到右上角的一个点，只要窗点在星矩内，就可以认为可以看到这颗星星。

那么，事情就变得简单明了了些，类似于扫描线，不过这次要得出的是一条x轴上权值最大的一个点了。

这就需要我们之前说的区间最大值的求法，可以类似上题，需要注意的是要离散化，可见我过去的博客~

以下代码！
```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define str string
#define db    double
#define DG(n)    cout<<n<<endl;
#define re(n)    n=read();
#define F(n)    for(int i=1;i<=n;i++)
#define li    i&-i
#define nel    (rt<<1)
#define ner    (rt<<1|1)
#define lson    l,r,nel
#define rson    l,r,ner
#define ls    l,mid
#define rs    mid+1,r
#define trm    ((tr[rt].l+tr[rt].r)>>1)
#define trf    ((tr[rt].r-tr[rt].l))

const int dx[5]={0,1,-1,0,0};
const int dy[5]={0,0,0,1,-1};
const int INF=0x3f3f3f3f; 
const db eps=1e-10;
const int LXB=2e6+1e5;
const db e=2.718281828459;
const db pai=3.1415926535;
const int lxb=3e3;

int n,m,k,l,t;
int rk[2*LXB],add[2*LXB];
int maxn = -1;


struct line{
    int x,yl,yh,f;
}le[LXB];


struct tree{
    int l,r,x;
}tr[LXB];

void pup(int rt){tr[rt].x = max(tr[ner].x , tr[nel].x);}

void build(int l,int r,int rt)
{
    tr[rt].r = r , tr[rt].l = l;
    if(l==r)return;
    int mid = trm;
    build(l,mid,nel);
    build(mid+1,r,ner);
}

void pdown(int rt)
{
    add[nel] += add[rt];
    add[ner] += add[rt];
    tr[nel].x += add[rt];
    tr[ner].x += add[rt];
    add[rt] = 0;
}


void up(int l,int r,int rt,int x)
{
    if(l <= tr[rt].l and tr[rt].r <= r){
        tr[rt].x += x ; add[rt]+=x ; 
        return ;
    }
    pdown(rt);
    int mid = trm;
    if(l<=mid)up(lson,x);
    if (r>mid)up(rson,x);
    pup(rt);
}

bool cmp(line l,line r)
{
    return  l.x!=r.x ? l.x<r.x : l.f>r.f; 
}



signed main(){
    ios::sync_with_stdio(0);cin.tie(0);
    int T = 1;
    cin>>T;
    while(T--){
        memset(tr,0,sizeof(tr));
        memset(add,0,sizeof(add));
        
        int w,h;
        cin>>n>>w>>h;
        int x0 , y0 , v;
        int cnt = 0,ans = 0;
        F(n) {
            cin >> x0 >> y0 >> v;
            le[(i<<1) - 1].x = x0, le[i<<1].x = x0+w-1;
            le[(i<<1) - 1].yh = le[i<<1].yh = y0+h-1;
            le[(i<<1) - 1].yl = le[i<<1].yl = y0;
            le[(i<<1) - 1].f = v, le[i<<1].f = -v;
            rk[++cnt] = y0;
            rk[++cnt] = y0+h-1;
        }
        sort(rk+1, rk+1+2*n);
        cnt = unique(rk+1, rk+2*n+1) - rk - 1;
        F(2*n){
            int p1 = lower_bound(rk+1, rk+cnt+1, le[i].yh) - rk;
            int p2 = lower_bound(rk+1, rk+cnt+1, le[i].yl) - rk;
            le[i].yh = p1;
            le[i].yl = p2;
        }
        sort(le+1, le+2*n+1, cmp);
        build(1, 2*n, 1);
        F(n*2){
            up(le[i].yl, le[i].yh, 1, le[i].f);
            ans = max(ans, tr[1].x);
        }
        DG(ans)
        end:;
    }
    return 0;//为，美，0
}

```