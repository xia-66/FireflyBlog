---
title: "关于数组中重复数据的处理"
published: 2024-03-13
updated: 2024-03-13
description: ""
tags: ["Js"]
category: "前端相关"
draft: false
---

**从数组中删除所有重复的元素**

    const arr = [1, 2, 3, 2, 4, 2, 5];
    
    const uniqueArr = arr.filter((item, index) => arr.indexOf(item) === index);
    
    console.log(uniqueArr); // 输出: [1, 2, 3, 4, 5]

使用filter()方法遍历数组中的每个元素，并对每个元素执行一个箭头函数，该函数检查元素在数组中的索引是否与其第一次出现的索引相同。如果索引相同，则将元素保留在新数组中

**从数组中删除所有重复的元素，包括对象和数组**

    const arr = [
      { a: 1, b: 2 },
      { a: 1, b: 2 },
      { a: 2, b: 3 },
      [1, 2],
      [1, 2],
      [2, 3],
    ];
    
    const uniqueArr = arr.filter((item, index, self) => {
      const itemString = JSON.stringify(item);
      return (
        index ===
        self.findIndex((element) => JSON.stringify(element) === itemString)
      );
    });
    
    console.log(uniqueArr);
    // 输出: [ { a: 1, b: 2 }, { a: 2, b: 3 }, [ 1, 2 ], [ 2, 3 ] ]
使用filter()方法遍历数组中的每个元素，并对每个元素执行一个箭头函数，该函数检查元素在数组中的索引是否与其第一次出现的索引相同。为了比较对象和数组，我们使用JSON.stringify()函数将它们转换为字符串