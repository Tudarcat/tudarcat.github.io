---
layout: post
title:  "Double Mean Method 实现随机红包或者RPG金币掉落"
date:   2026-03-01 17:00:00 +0800
categories: 算法 游戏 C++
---

当我们在某群聊发送红包时，或者在一个固定量金币的boss掉落随机金币袋的场景下，二倍均值法起到了重要作用。本文介绍这种随机算法。

# 定义

二倍均值法是一种用于随机分配资源的算法，旨在确保分配的公平性和随机性，同时避免因顺序导致的不公平问题。其核心思想是动态调整随机金额的范围，使每个人的期望值相等。

# 算法过程

假设第 $i$ 次分配时金币池含有 $M_i$ 元，有 $N_i$ 个人想要分配这些金币。此轮中，对应的编号为 $i$ 的人获取的金币数量落在 $[0, \frac{M_i}{N_i} \times 2]$ 区间中。假设随即取出了 $C_i$ 元，下一轮 $M_{i+1}=M_{i}-C_i$，$N_{i+1} = N_{i} - 1$.

# 简单证明

要证明公平性，即证明每一轮取出的金币的期望相等。为了简化，假设每一轮取出的是期望值。不妨设 $E_i$ 为每一轮的期望取出金币数。可以得到 $$E_1= \frac{M_1}{N_1}$$$$E_2= \frac{M_1}{N_1}=\frac{M_1-E_1}{N_1-1}=\frac{M_1}{N_1}\times\frac{N_1-1}{N_1-1}=E_1$$以此类推，$E_i=E_3=E_2=E_1$所以全部轮的期望相等。

# 算法实现

```cpp
// assume that one player get at least 1 point
vector<int> redPacket(int totalAmount, int peopleCount) {
    vector<int> result;
    if (totalAmount < peopleCount * 1) {
        cout << "Not enough points" << endl;
        return result;
    }

    srand((unsigned)time(NULL));

    int remainMoney = totalAmount;
    int remainPeople = peopleCount;

    for (int i = 0; i < peopleCount - 1; ++i) {
        int max = (remainMoney / remainPeople) * 2;
        // [1, max]
        int amount = rand() % max + 1;

        if (amount > remainMoney) {
            amount = remainMoney;
        }

        result.push_back(amount);
        remainMoney -= amount;
        --remainPeople;
    }

    // the last player get left points
    result.push_back(remainMoney);

    return result;
}
```

对于固定的奖池，每一轮让某一个人去抽取随机金币。由上文，每个人得到的金币期望是完全相同的。不过由于每一轮我们都假设至少得到一个金币，所以实际应用时略有误差。不过对于金币足够多的场景下，得到金币的多少完全只是个人的运气问题。

另外，本文代码的实现仅作为参考，它是非常不安全的，且受限于C语言 $rand()$ 的编译器实现，最高的可取到的随机数为 ```RAND_MAX = 32767```。