---
layout: post
title:  "敏感词过滤及游戏秘籍的算法-AC自动机"
date:   2026-03-01 17:00:00 +0800
categories: 算法 游戏 C#
---

在电子游戏中，尤其是能够进行联机交互的电子游戏中，脏话屏蔽、秘籍、Combo连招等非常常见。本文介绍了一种能够高效实现这三个目标的算法。


# AC自动机定义

**AC自动机**（Aho-Corasick Automaton）是一种有限状态自动机，由Alfred V. Aho和Margaret J. Corasick于1975年提出。它是一种**多模式匹配算法**的核心数据结构，能够在一次扫描文本的情况下，同时查找多个模式串（关键词）的所有出现位置。

AC自动机本质上是在**Trie树（字典树）** 的基础上，增加了**失败指针（fail指针）**，使得当当前路径匹配失败时，能够快速转移到另一个具有最长公共前缀的路径继续匹配，从而避免回溯，实现线性时间复杂度的多模式匹配。

# 结构

AC自动机的基于一棵**Trie树**。此外，AC自动机对每个节点都额外加入了**fail**指针，类似于KMP的next数组，fail指针为当前节点提供了失配时的一条回溯捷径。

对于每一个节点 $u \in Q$ ，$u$ 的 $fail$ 指针指向了 $v$ ，其中 $v$ 是 $u$ 的最长后缀。


# 构造

## 1.Trie树的构造

在构造失败指针之前，我们先将所有模式串的Trie树构造出来。这一部分的构造比较简单，就是一棵标准的Trie树。
```csharp
using System;
using System.Collections.Generic;
using System.Text;

public class AhoCorasick
{
    private Dictionary<int, Dictionary<char, int>> trie;
    private int[] fail;
    private int[] end;
    private int cnt;
    private HashSet<char> charset;

    public AhoCorasick()
    {
        trie = new Dictionary<int, Dictionary<char, int>>();
        fail = new int[100005];
        end = new int[100005];
        cnt = 0;
        charset = new HashSet<char>();
        trie[0] = new Dictionary<char, int>();
        Array.Fill(fail, 0);
        Array.Fill(end, 0);
    }

    // 插入模式串
    public void Insert(string s)
    {
        int cur = 0;
        foreach (char c in s)
        {
            charset.Add(c); // 记录出现的字符集
            if (!trie[cur].ContainsKey(c))
            {
                trie[++cnt] = new Dictionary<char, int>();
                trie[cur][c] = cnt;
                end[cnt] = 0;
            }
            cur = trie[cur][c];
        }
        end[cur]++;
    }
```


## 2.构造失败指针

我们先令 $root$ 的 $fail$ 指针指向自己， 且对于 $root$ 的第一层儿子，他们的 $fail$ 指针也指向 $root$. 我们通过 $bfs$ 从上到下一层层地构造失败指针。首先将 $root$ 及其儿子入队，然后对于每一个队伍中的元素 $u$ ，其儿子 $v=next(u,i)$ .如果 $v$ 未定义，直接将 $next(u, i) = next(fail(u), i)$ 。如果 $v$ 存在，$fail(v)=next(fail(u),i)$ .

我们可以递推地证明此算法地正确性。

**最开始** ：$root$ 及其儿子指向了 $root$ 本身，$root$即$empty$ 是单个字符或者空字符地最长后缀，合理。

**转移**： 显然 $u$ 的$fail(u)$ 指向了 $u$ 的最长后缀，那么对于 $v=next(u,i)$，在 $fail(u)$ 的基础上转移同一个字符，一定还是 $v$ 的最长后缀。

**最终**：所有的节点的 $fail$ 都指向了正确的节点。构造算法结束。

 ```csharp
// 构建失配指针
public void Build()
{
    Queue<int> q = new Queue<int>();
    fail[0] = 0;

    // 初始化第一层
    foreach (var pair in trie[0])
    {
        char c = pair.Key;
        int child = pair.Value;
        fail[child] = 0;
        q.Enqueue(child);
    }

    // BFS构建失配指针
    while (q.Count > 0)
    {
        int u = q.Dequeue();
        foreach (var pair in trie[u])
        {
            char c = pair.Key;
            int v = pair.Value;
            
            // 如果v存在
            fail[v] = trie[fail[u]].ContainsKey(c) ? trie[fail[u]][c] : 0;
            q.Enqueue(v);
        }
        
        // 路径压缩：处理不存在的边
        foreach (char c in charset)
        {
            if (!trie[u].ContainsKey(c))
            {
                trie[u][c] = trie[fail[u]].ContainsKey(c) ? trie[fail[u]][c] : 0;
            }
        }
    }
}
 ```


# 使用

我们根据**Trie树**的转移方法，对字符进行转移。当无法向下方转移时，我们通过失败指针转移到$fail(u)$ .

```csharp
// 查询：统计文本串中出现多少个模式串
public long Query(string text)
{
    int cur = 0;
    long ans = 0;
    foreach (char c in text)
    {
        if (charset.Contains(c))
        {
            cur = trie[cur][c];
        }
        else
        {
            // 如果字符不在字符集中，回到根节点
            cur = 0;
            continue;
        }
        
        // 沿着fail链累加所有匹配的模式串
        int tmp = cur;
        while (tmp != 0)
        {
            ans += end[tmp];
            // 如果不需要重复计算，可以加上这行防止重复
            // end[tmp] = 0;
            tmp = fail[tmp];
        }
    }
    return ans;
}

// 获取所有匹配的位置
public List<(int position, string pattern)> FindAllMatches(string text, string[] patterns)
{
    var patternSet = new HashSet<string>(patterns);
    
    int cur = 0;
    var results = new List<(int, string)>();
    
    for (int i = 0; i < text.Length; i++)
    {
        char c = text[i];
        if (charset.Contains(c))
        {
            cur = trie[cur][c];
        }
        else
        {
            cur = 0;
            continue;
        }
        
        int tmp = cur;
        while (tmp != 0)
        {
            if (end[tmp] > 0)
            {
                results.Add((i, "Matched pattern ending at position " + i));
            }
            tmp = fail[tmp];
        }
    }
    return results;
}

```

使用示例

```csharp
class Program
{
    static void Main(string[] args)
    {
        AhoCorasick ac = new AhoCorasick();
        
        ac.Insert("hello");
        ac.Insert("world");
        ac.Insert("he");
        ac.Insert("llo");
        ac.Insert("123");
        
        ac.Build();
        
        // 查询文本
        string text = "hello world, this is a test 123";
        long count = ac.Query(text);
        Console.WriteLine($"Found {count} matches");
        
        var matches = ac.FindAllMatches(text, new string[] { "hello", "world", "he", "llo", "123" });
        foreach (var match in matches)
        {
            Console.WriteLine($"Pattern matched at position {match.position}: {match.pattern}");
        }
    }
}
```

此算法可以快速的匹配多个模式串，从而实现在一个字符串中高效的查找所有可能的“脏话”并进行处理。

此外，我们可以实现类似于 GTA VC 的秘籍系统。当我们按下了某一个按键，实际上触发了一次转移。当我们转移到某个 $end$ 时，就可以根据 $end$ 所在节点记录的信息触发某个秘籍效果。