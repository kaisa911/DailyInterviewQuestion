# 寻找数组中是否含有相同的数字元素

## 两个循环寻找重复元素

```javascript
var containsDuplicate = function (nums) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] === nums[i]) return true;
    }
  }
  return false;
};
```

这种方案几乎是最大的时间复杂度，所以并不推荐使用这个方案。

## 数组排序后两两比对

```javascript
var containsDuplicate = function (nums) {
  nums = nums.sort((a, b) => a - b);
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] === nums[i + 1]) return true;
  }
  return false;
};
```

这个方法比较兼顾时间复杂度和空间复杂度

## 构造 Set 数组比对

这个方法是最为照顾时间复杂度的

```javascript
var containsDuplicate = function (nums) {
  const set = Array.from(new Set(nums));

  return nums.length !== set.length;
};
```

## leetcode 运行时间表

| 方法号 | 运行时间 | 使用空间 |
| ------ | -------- | -------- |
| 1      | 1224 ms  | 37 MB    |
| 2      | 96 ms    | 38.2 MB  |
| 3      | 60 ms    | 42.9 MB  |
