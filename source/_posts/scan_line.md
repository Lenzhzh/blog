---
title: 扫描线模板
date: 2023-6-14 21:17:12
tags: 
    - 算法
    - 线段树
    - 扫描线
description: |
    最近刚写了一道题（指6-14），对扫描线有了一些感悟。
    这道题本来也刷过一遍，但因为当时不求甚解。
    现在二刷就熟悉了很多。所以有些时候好用的东西可以反复...(龙叔DNA错乱)

---
# 扫描线！——

首先，扫描线，顾名思义，是拿一条线来扫描，然后得出总面积。

在这道题中，我们将x设为轴，然后从x的左端一直扫描到右端，每次扫描算一段长度，把长度加起来就是我们的面积总和了。

![](./images/unflocing_pic/scan_line_1.gif)

# 具体实现！——

对于x轴上的y轴线，我们采用线段树计算所有的长度。对于 $[1,maxn]$ 的范围，用线段树求出一条线上的覆盖总面积。每一个矩形在一条线上可以视为一个$[x1,x2]$范围中所有的值改为1，而y轴可以视为一颗线段树。

那么，当我们扫过一条之后，又该怎么办呢？

我们对每个矩形进行拆分，将其存为两条线，一条左边界和一条右边界，每条线有yh，yl，cnt，f这3个值。

```cpp
struct line{
    int x,yl,yh,f;//分别表示x坐标，最高的y值，最低的y值和是否为左边界，左边界为1，右边界为-1
}le[LXB];

```

如果我们的line.f为1，就对范围+1，如果为-1，就说明这个矩形不需要再计算了，就将范围-1。这样在中间过程中就不用在意了，也有一种类似前缀和的思想在。

这样...这道题就解决...了吗！

那还是太小看这道蓝题了，接下来是...

# 离散化！

注意到x的范围是 $[1,1 /times 10^9]$，但$n$的范围是 $[1,1 /times 10^6]$，我们对线进行离散化。

那对y的离散化要怎么做呢？

很简单，我们知道，一个x坐标上涵盖的y值一定在给定的几个y中间。所以只要记住被排序之后的y坐标，然后将相邻的y坐标记为一个小格，最后算小格的总长度就可以了。当然，区别于普通线段树，pushup就要改良一下了。


```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define str string
#define db    double
#define DG(n)    cout<<n<<endl;
#define re(n)    n=read();
#define F(n)    for(int i=1;i<=n;i++)
#define F1(n)    for(int i=n;i>=1;i--)
#define F2(n,s)    for(int i=1;i<=n;i+=s)
#define F4(n,m)    for(int i=1;i<=n;i++){for(int j=1;j<=m;j++)
#define Ftr1    for(int i=1;i<=n;i+=li)tr[i]+=x;
#define Ftr2    for(int i=t;i>=1;i-=li)ans+=tr[i];
#define FD(name,n)    for(int name=1;name<=n;name++)
#define li    i&-i
#define nel    (rt<<1)
#define ner    (rt<<1|1)
#define lson    l,r,nel
#define rson    l,r,ner
#define ls    l,mid
#define rs    mid+1,r
#define trm    ((tr[rt].l+tr[rt].r)>>1)
#define trf    ((tr[rt].r-tr[rt].l))
#define dip    pair<db,int>
#define dii     pair<int,int>
#define mp(a,b)    make_pair(a,b)
#define op(a,type)    bool operator a (const type &an)

const int dx[5]={0,1,-1,0,0};
const int dy[5]={0,0,0,1,-1};
const int INF=0x3f3f3f3f; 
const db eps=1e-10;
const int LXB=2e6+1e5;
const db e=2.718281828459;
const db pai=3.1415926535;
const int lxb=3e3;

int n,m,k,l,t;
int rk[2*LXB],val[2*LXB];
int maxn = -1;


struct tree{
    int l,r;
    int cnt,len;//cnt记录一个地块被覆盖的次数，如果有被覆盖就做一次累加，没有就更新节点。
}tr[LXB];

struct line{
    int x,yl,yh,f;
}le[LXB];


void pup(int rt)
{
    if (tr[rt].cnt) {tr[rt].len = val[tr[rt].r+1] - val[tr[rt].l];}//地块被覆盖，此时长度就等于节点的右边值减去左边值。此处val数组表示离散化前排序的节点的长度。
    //注意如果存在cnt，这块地块中一定被全部覆盖了，所以直接等于区间长是合理的。
    else {tr[rt].len = tr[nel].len+tr[ner].len;}//若未被完全覆盖，则值等于两个子节点的和。
}

void build(int l,int r,int rt)
{
    tr[rt].l = l, tr[rt].r = r;
    if (l == r)return ;
    int mid =  trm;
    build(l, mid, nel);
    build(mid+1 , r , ner);
}

void add(int l, int r, int rt, int x)
{
    if (l <= tr[rt].l and tr[rt].r <= r)
    {
        tr[rt].cnt += x; pup(rt); return ;
    }//如果被全覆盖，则标记
    int mid = trm;
    if (l <= mid) add(l, r, nel, x);
    if (r > mid) add(l, r, ner ,x);
    pup(rt);//更新节点
}

bool cmp(line l,line r)
{
    return  l.x!=r.x ? l.x<r.x : l.f>r.f; 
}



signed main(){
    ios::sync_with_stdio(0);cin.tie(0);
    int T = 1;
    
    while(T--){
        cin>>n;
        int x0, x2, y0 ,y2;
        int cnt = 0,ans = 0;
        F(n) {
            cin >> x0 >> y0 >> x2 >> y2;
            le[(i<<1) - 1].x = x0, le[i<<1].x = x2;
            le[(i<<1) - 1].yh = le[i<<1].yh = y2;
            le[(i<<1) - 1].yl = le[i<<1].yl = y0;
            le[(i<<1) - 1].f = 1, le[i<<1].f = -1;
            rk[++cnt] = y0;
            rk[++cnt] = y2;
        }//读入结构体，一个矩形有两个边
        sort(rk+1, rk+1+2*n);
        cnt = unique(rk+1, rk+2*n+1) - rk - 1;
        F(2*n){
            int p1 = lower_bound(rk+1, rk+cnt+1, le[i].yh) - rk;
            int p2 = lower_bound(rk+1, rk+cnt+1, le[i].yl) - rk;
            val[p1] = le[i].yh;
            val[p2] = le[i].yl;
            le[i].yh = p1, 
            le[i].yl = p2;
        }//两部离散化。x也可以离散化掉，之后计x直接看作一个宽度一定的小矩形就好了
        sort(le+1, le+2*n+1, cmp);
        build(1, 2*n, 1);
        F(n*2-1){
            add(le[i].yl, le[i].yh-1, 1, le[i].f);//枚举每条线所在位置。
            //注：之所以对最大的y值-1，是因为线段树里的每个点的问题。覆盖的是从yl到yh的下一个格点，所以-1
            ans += tr[1].len * (le[i+1].x - le[i].x);//每条线视为一个小矩形
        }
        cout<<ans<<endl;
        end:;
    }
    return 0;//为美好的世界献上return 0
}

```
