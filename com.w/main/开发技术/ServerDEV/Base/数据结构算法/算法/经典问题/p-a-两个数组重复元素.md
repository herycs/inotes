# 两个数组重复元素

## 策略

方案一：使用两个for循环遍历两个数组，时间复杂度为O(),空间复杂度O(1)

方案二：将两个数组排好序，之后设置两个数组的下标i=0,j = 0,从头到尾扫描两个数组中数是否相同，如果不同，较小的数靠后移位，如果相同，则两个下标同时后移，直到其中一个下标到了末尾，则结束。时间复杂度为O(),空间复杂度O(1)

方案三：对长度较短的数组建立hash表，然后利用较长的数组在hash表中查找是否存在。时间复杂度为O(1),空间复杂度O(min(N,M))

## 排序 + 指针

O(n + m)

```c
while (index1 < len1 && index2 < len2) {
	if (nums1[index1] < nums2[index2]) {
		index1++;
	} else if (nums1[index1] > nums[index2]) {
		index2++;
	} else {
		res.add(nums[index1]);
		index1++;
		index2++;
	}
}
```

## 短数组 + 二分

短数组为查找基准

