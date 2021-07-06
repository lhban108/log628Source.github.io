---
layout: Set Map
title: Set Map
date: 2019-11-30 14:46:37
tags: 算法
keywords: 博客文章密码
## password: 123123
message:  输入密码，查看文章
---

## 一、集合(Set)是什么

- 一种**无序且唯一**的数据结构
- ES6中有集合，名为Set
- 集合的常用操作：去重、判断某元素是否在集合中、求交集...
  
([MDN-Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set))

`Set`对象是值的集合，你可以按照插入的顺序迭代它的元素。 Set中的元素只会**出现一次**，即 `Set` 中的元素是唯一的。

```javascript
// 创建一个set集合
let mySet = new Set([1, {a: 1}, 3]); // return undefined => [1, {a: 1}, 3]
let mySet2 = new Set(); // return undefined => []

// 增加元素
mySet.add(1); // return [1, {a: 1}, 3]
mySet.add(4); // return [1, {a: 1}, 3, 4]

// 删除某个元素
mySet.delete(3); // true  => [1, {a: 1}, 4]

// 清空
mySet2.clear(); // return undefined => []

// 判断存在与否
mySet.has(4); // reruen true

// 获取长度 (长度为实例属性，不是方法)
mySet.size; // 3

// 迭代
for (let item of mySet) console.log(item); // 1 {a: 1} 3

mySet.forEach(item => { console.log(item) }); // 1 {a: 1} 3

for (let item of mySet.keys()) console.log(item); // 1 {a: 1} 3

for (let item of mySet.values()) console.log(item); // 1 {a: 1} 3

for (let [key, value] of mySet.entries()) console.log(key, value); // 1 1  {a: 1} {a: 1} 3 3

for (let [key, value] of mySet.entries()) console.log(key === value); // true true true

// 与数组相关
let arr1 = Array.from(mySet);
let arr2 = [...mySet]

let mySet3 = new Set(arr1);
```

## 二、字典(Map)是什么

- 与集合类似，字典也是一种存储唯一值的数据结构，但是它以**键值对**的形式来存储
- ES6中有字典，名为Map
- 字典的常用操作：键值对的增删改查

([MDN-Map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/map))

`Map` 对象保存键值对，并且能够记住键的原始插入顺序。任何值(对象或者原始值) 都可以作为一个键或一个值。

```javascript
// 创建一个map
let myMap = new Map();

let keyString = 'a string';
let keyObj = {};
let keyFunc = function() {};

// 添加
myMap.set(keyString, '和键"a string"关联的值');
myMap.set(keyObj, "和键keyObj关联的值");
myMap.set(keyFunc, "和键keyFunc关联的值");
myMap.set(NaN, "not a number");

// 获取长度 (长度为实例属性，不是方法)
myMap.size;

// 读取值
myMap.get(keyString);    // "和键'a string'关联的值"
myMap.get(keyObj);       // "和键keyObj关联的值"
myMap.get(keyFunc);      // "和键keyFunc关联的值"
myMap.get(NaN); // "not a number"

myMap.get('a string');   // "和键'a string'关联的值"
                         // 因为keyString === 'a string'
myMap.get({});           // undefined, 因为keyObj !== {}
myMap.get(function() {}); // undefined, 因为keyFunc !== function () {}

// 遍历
myMap.forEach((value, key) =>{
  console.log(key + " = " + value);
})

for (let [key, value] of myMap) {
  console.log(key + " = " + value);
}

for(let a of myMap.keys()) console.log(a);

for(let a of myMap.values()) console.log(a)

// map与数组的关系
let arr1 = [["key1", "value1"], ["key2", "value2"]];
let myMap2 = new Map(arr1); // 数组 转 map
let arr2 = arr1.from(myMap2); // map 转 数组
let arr3 = [...myMap2]; // map 转 数组

// 发现一个问题
let otherNaN1 = Number("foo1");
let otherNaN2 = Number("foo2");

myMap.get(otherNaN1); // "not a number"
myMap.get(otherNaN2); // "not a number"


otherNaN1 == otherNaN2; // false
otherNaN1 === otherNaN2; // false
myMap.get(otherNaN1) === myMap.get(otherNaN2); // true
// why ??
```

## 三、练习题

### 1、两个数组的交集-1

> ([leecode-349](https://leetcode-cn.com/problems/intersection-of-two-arrays/))

题目描述：给定两个数组，编写一个函数来计算它们的交集。

示例1：

```text
输入：nums1 = [1,2,2,1], nums2 = [2,2]
输出：[2]
```

示例2：

```text
输入：nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出：[9,4]
```

```text
说明：
- 输出结果中的每个元素一定是唯一的。
- 我们可以不考虑输出结果的顺序。
```

```javascript
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number[]}
 */
var intersection = function(nums1, nums2) {
    // 解法一: Set
    return [...new Set(nums1)].filter(item => nums2.includes(item));

    // 解法二: Map
    // const map = new Map();
    // nums1.forEach(item => {
    //     map.set(item, true);
    // })
    // const res = [];
    // nums2.forEach(item => {
    //     if (map.get(item)) {
    //         res.push(item);
    //         map.delete(item)
    //     }
    // })
    // return res;
};
/* Set
时间复杂度： O(n²)  （首先num1的filter遍历，所以复杂度为O(n)；num2的includes也遍历，所以时间复杂度为O(n)，嵌套循环后就是 n*n，即 O(n²)
     更严谨的说，时间复杂度为  O(m*n), 其中m为num1去重后的长度，n为num2的长度
空间复杂度： 除了num1和num2数组内部的元素，没有新增变量，所以空间复杂度为 O(m), 其中m是去重后num1 的长度
*/
/*Map
时间复杂度： O(m+n); 其中m是nums1的长度，n是num2的长度
空间复杂度： O(m); 其中m是nums1的长度
*/
```

### 2、两个数组的交集-2

> ([leecode-350](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/))

题目描述：给定两个数组，编写一个函数来计算它们的交集。

示例1：

```text
输入：nums1 = [1,2,2,1], nums2 = [2,2]
输出：[2,2]
```

示例2：

```text
输入：nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出：[4,9]
```

```text
说明：
- 输出结果中每个元素出现的次数，应与元素在两个数组中出现次数的最小值一致。
- 我们可以不考虑输出结果的顺序。
```

```text
进阶：
- 如果给定的数组已经排好序呢？你将如何优化你的算法？
- 如果 nums1 的大小比 nums2 小很多，哪种方法更优？
- 如果 nums2 的元素存储在磁盘上，内存是有限的，并且你不能一次加载所有的元素到内存中，你该怎么办？
```

```javascript
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number[]}
 */
var intersection = function(nums1, nums2) {
    // 解法一: Set
    return [...new Set(nums1)].filter(item => nums2.includes(item));

    // 解法二: Map
    // const map = new Map();
    // nums1.forEach(item => {
    //     map.set(item, true);
    // })
    // const res = [];
    // nums2.forEach(item => {
    //     if (map.get(item)) {
    //         res.push(item);
    //         map.delete(item)
    //     }
    // })
    // return res;
};
/* Set
时间复杂度： O(n²)  （首先num1的filter遍历，所以复杂度为O(n)；num2的includes也遍历，所以时间复杂度为O(n)，嵌套循环后就是 n*n，即 O(n²)
     更严谨的说，时间复杂度为  O(m*n), 其中m为num1去重后的长度，n为num2的长度
空间复杂度： 除了num1和num2数组内部的元素，没有新增变量，所以空间复杂度为 O(m), 其中m是去重后num1 的长度
*/
/*Map
时间复杂度： O(m+n); 其中m是nums1的长度，n是num2的长度
空间复杂度： O(m); 其中m是nums1的长度
*/
```

### 2、有效的括号

> ([leecode-20](https://leetcode-cn.com/problems/valid-parentheses/))

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
  
有效字符串需满足：
  
左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
注意空字符串可被认为是有效字符串。  

示例 1:

```text
输入: "()"
输出: true
```

示例 2:

```text
输入: "()[]{}"
输出: true
```

示例 3:

```text
输入: "(]"
输出: false
```

示例 4:

```text
输入: "([)]"
输出: false
```

示例 5:

```text
输入: "{[]}"
输出: true
```

```javascript
/**
 * @param {string} s
 * @return {boolean}
 */
/**
 * 时间复杂度:  O(n)
 * 空间复杂度:  O(1)
 */
var isValid = function(s) {
    if (!s) return true;
    if (s.length & 2 === 1) return false; 
    let map = new Map();
    map.set('(', ')');
    map.set('[', ']');
    map.set('{', '}');
    const stack = [];
    for(let i = 0; i < s.length; i++) {
        if (map.has(s[i])) {
            stack.push(s[i]);
        } else if (map.get(stack.pop()) !== s[i]) {
            return false
        }
    }
    return stack.length === 0;
};
/**
 * 时间复杂度: O(n) n为s的长度
 * 空间复杂度: O(n) 因为map一个常量级的O(1)的空间复杂度
 * /
```

### 3、两数之和

> ([leecode-1](https://leetcode-cn.com/problems/two-sum/))

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
  
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。
  
示例:

```text
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    // 解法一： map 时间复杂度: O(n) 空间复杂度: O(n);
    const map = new Map();
    for(let i = 0; i < nums.length; i++) {
        const n1 = nums[i];
        const n2 = target - n1;
        if (map.has(n2)) {
            return[map.get(n2), i];
        } else {
            map.set(n1, i);
        }
    }

    // 解法二：  时间复杂度: O(n) 空间复杂度: O(1);
    // for(let i = 0; i < nums.length; i++) {
    //   if (nums.indexOf(target - nums[i]) > -1 && nums.indexOf(target - nums[i]) !== i) {
    //     return [i, nums.indexOf(target - nums[i])]
    //   }
    // }
};
```

### 4、无重复字符的最长子串

> ([leecode-3](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/))

给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。
  
示例1:

```text
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

示例2:

```text
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

示例3:

```text
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

示例4:

```text
输入: s = ""
输出: 0
```

```javascript
/**
 * @param {string} s
 * @return {number}
 * 解题思路：
 * 1、用双指针维护一个滑动窗口，从来剪切子串
 * 2、不断移动右指针，遇到重复字符串，就把左指针移动到重复字符串的下一位
 * 3、移动右指针的过程中，记录所有窗口的长度，并返回最大值
 */
var lengthOfLongestSubstring = function(s) {
    let l = 0;
    let res = 0;
    const map = new Map();
    for (let r = 0; r < s.length; r++) {
        if (map.has(s[r]) && map.get(s[r]) >= l) {
            l = map.get(s[r]) + 1;
        }
        map.set(s[r], r);
        res = Math.max(res, r - l + 1);
    }
    return res;
};
/**
 * 时间复杂度： O(n)
 * 空间复杂度： O(m)  m是字符串s不重复字符的个数
 */
```

### 5、最小覆盖子串

> ([leecode-76](https://leetcode-cn.com/problems/minimum-window-substring/))

给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。

**注意：如果 s 中存在这样的子串，我们保证它是唯一的答案。**
  
示例1:

```text
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
```

示例2:

```text
输入：s = "a", t = "a"
输出："a"
```

提示：

- 1 <= s.length, t.length <= 105
- s 和 t 由英文字母组成

```javascript
/**
 * @param {string} s
 * @param {string} t
 * @return {string}
 */
/**
 * 用双指针维护一个滑动窗口
 * 移动右指针，直到找到包含t的所有子串，此时再移动左指针，尽量减少包含t的子串的长度。
 * 时间复杂度:  O(m + n) ,m是t的长度,n是s的长度
 * 空间复杂度:  O(m) ,m是t中不同字符的个数
 */
var minWindow = function(s, t) {
    let l = 0;
    let r = 0;
    const need = new Map();
    for(const c of t) {
        need.set(c, need.get(c) ? need.get(c) + 1 : 1);
    }
    let needType = need.size;
    let resStr = '';
    while(r < s.length) {
        const c1 = s[r];
        if (need.has(c1)) {
            need.set(c1, need.get(c1) - 1);
            if (need.get(c1) === 0) needType -= 1;
        }
        // 当needType=0时，表示右指针已经移动到可以包含t的位置，此时需要移动左指针
        while(needType === 0) {
            if (!resStr || resStr.length > s.substring(l, r + 1).length) {
                resStr = s.substring(l, r + 1);
            }
            const c2 = s[l];
            if (need.has(c2)) {
                need.set(c2, need.get(c2) + 1);
                if (need.get(c2) === 1) needType += 1;
            }
            l += 1;
        }
        r += 1;
    }
    return resStr;
};
```
