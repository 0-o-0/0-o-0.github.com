---
layout: post
title: "Fenwick Tree"
description: ""
category: Data Structure
tags: []
---
{% include JB/setup %}


[Fenwick tree(or binary indexed tree)](http://en.wikipedia.org/wiki/Fenwick_tree) is an ingenious data structure.

It maintain a tree on the array with the following characteristics:

1. Calculate the sum in an interval within `O(log n)`
2. Modify an element within `O(log n)`
3. Require only `O(n)` memory

## 1. Point Update, Range Query

First, we can reduce range query to prefix sums. That is, `Sum(a..b) = Sum(1..b) - Sum(1..a-1)`.

So the remaining question is build a data structure to support modifying an element and querying prefix sum in `O(log n)`.

The basic idea of Fenwick tree is just as an integer is the sum of appropriate powers of two, so can a prefix sum be represented as the appropriate sum of sets of cumulative 'subsums'. Figure 1 shows a table of size 16.

![alt text](/assets/pictures/subsets.png "The Tree of subsets")

Precisely, `tree[idx]` is the sum of elements from index `idx - 2^r + 1` to index `idx`, where `r` is the position of the last digit 1 in `idx`.

Following are code samples:

{% highlight cpp %}

// query the sum in interval [l, r]
int query(int l, int r)
{
    return query(r) - query(l-1);
}

// query the prefix sum of 'idx'
int query(int idx)
{
    int ret = 0;
    while (idx > 0)
    {
        ret += tree[idx];
        idx -= idx & -idx;
    }
    return ret;
}
    
// add the element of 'idx' with 'val'
void update(int idx, int val)
{
    while (idx <= length)
    {
        tree[idx] += val;
        idx += idx & -idx;
    }
}

{% endhighlight %}

_Note: the index of tree should begin at 1_

You may notice this expression `idx & -dx`, it yields the last digit 1 of idx. Interesting, right? What's it under the hood?

    decimal                 binary
    -------------------------------------------
          8    00000000000000000000000000001000
         -8    11111111111111111111111111111000
       8&-8    00000000000000000000000000001000
    -------------------------------------------
         10    00000000000000000000000000001010
        -10    11111111111111111111111111110110
     10&-10    00000000000000000000000000000010

First, how to get the binary digits of -x, if we have the binary digits of x already?

Yes, two's complement of the value and add 1.

So the continuous 0s at the tail of x will be changed to continuous 1s and then be added up to previous digit. This digit is the only one same with original x's digits.

An alternative of isolating last digit is `idx - (idx & (idx - 1))`.

## 2. Range Update, Point Query

We store the difference between current value and original value.

For example, adding the elements between i and j with 1. If we check the difference between adjacent elements, we can easily get the following image.

![alt text](/assets/pictures/range_update.gif "Range update")

Let's denote the difference array as f. So the element at idx is Sum(f[1]..f[idx]).


## 3. Range Update, Range Query

We consider update(a, b, c) as add the elements between a and b with c.

The add array A looks like 0 0 0 ... c c c ... 0 0 0.

Then consider the difference array B, it looks like ... 0 0 c 0 0 ... 0 0 -c 0 0...

We can see for every update, only 3 of the array values of B[1..n] are changing. So we will be creating a Fenwick tree for B[1..n] so that we will be updating it in `O(log n)` time for each update.

Let us now consider the query. The value of A[i] will be the sum of values of B[j] for all 1 <= j <= i. Therefore sum of (A[i] for all x <= i <= y) = (y - x + 1) * sum of (B[j] for all 1 <= j <= x) + (y + 1) * sum of (B[j] for x < j <= y) - sum of (j * B[j] for all x < j <= y). To obtain sum of (j * B[j] for all x < j <= y), we need to create another Fenwick tree C[1..n] such that C[k] = k * B[k].

[poj3468](http://poj.org/problem?id=3468) and [spoj PYRSUM2](http://www.spoj.com/problems/PYRSUM2/) are places to test your code.

## 4. Point Update, Range Minimum Query

This is an original solution to this problem. Previously, the classical solution is to use segment tree with lazy propagaion.

In the solution we use two arrays (which are similar to Fenwick tree's array) left[] and right[] to store data.

left[i] stores the minimum in interval [i - 2^r + 1, i]. right[i] stores the minimum in interval [i, i + 2^r - 1].

The following image shows their responsibilities.

In the left/right array, the i-th element covers 2^r elements in original array.

{% highlight cpp %}

void build()
{
    for (int i = 1; i <= n; ++i)
    {
        left[i] = right[i] = val[i];
    }
    for (int j = 1; two(j) <= n; ++j)
    {
        for (int i = two(j); i <= n>>j<<j; i += two(j+1))
        {
            left[i] = max(left[i], max(left[i-two(j-1)], right[i-two(j-1)]));
            right[i] = max(right[i], max(left[i+two(j-1)], right[i+two(j-1)]));
        }
    }
}

void update(int idx, int nv)
{
    val[idx] = nv;
    int up = idx, down = idx;
    if (idx&1)
    {
        left[idx] = right[idx] = nv;
        up += lowbit(idx), down -= lowbit(idx);
    }


    while (true)
    {
        bool goup=false;
        if (up > n)
        {
            if (down == 0) break;
            goup = false;
        }
        else
        {
            if (down == 0) goup = true;
            else goup = lowbit(down) > lowbit(up);
        }
        if (!goup)
        {
            int mid = down + (lowbit(down)>>1);
            if (mid <= n)
            {
                right[down] = max(val[down], max(right[mid], left[mid]));
            }
            down -= lowbit(down);
        }
        else
        {
            int mid = up - (lowbit(up)>>1);
            left[up] = max(val[up], max(left[mid], right[mid]));
            up += lowbit(up);
        }
    }
}

int query(int a, int b)
{
    int i, ni, ret = 0;

    for (i = a, ni = i + lowbit(i); ni<= b; i = ni, ni += lowbit(ni))
        ret = max(ret, right[i]);

    ret = max(ret, val[i]);

    for (i = b, ni = i - lowbit(i); ni>= a; i = ni, ni -= lowbit(ni))
        ret = max(ret, left[i]);

    return ret;
}

{% endhighlight %}

[hdoj1754](http://acm.hdu.edu.cn/showproblem.php?pid=1754) is a place to test your code.


## 5. 2D Fenwick tree

{% highlight cpp %}

void update(int x, int y, int val)
{
    while (x <= max_x)
    {
        int ty = y;
        while (ty <= max_y)
        {
            tree[x][ty] += val;
            y += lowbit(y);
        }
        x += lowbit(x);
    }
}

{% endhighlight %}

[poj2155](http://poj.org/problem?id=2155) and [ural1470](http://acm.timus.ru/problem.aspx?space=1&num=1470) are places to test your code.


## References

[Topcoder Algo Tutorial -- Binary Indexed Trees](http://community.topcoder.com/tc?module=Static&d1=tutorials&d2=binaryIndexedTrees)

[habrahabr -- Fenwick tree for maximum support](http://habrahabr.ru/post/160099/)
