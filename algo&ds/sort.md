O(n^2^)

O(nlogn)

O(n)

O(1)

## 选择排序

选择排序的思想是：双重循环遍历数组，每经过一轮比较，找到最小元素的下标，将其交换至首位。

- 冒泡排序法是稳定的，选择排序法是不稳定的。

当要排序的内容是一个对象的多个数字属性，且其原本的顺序存在意义时，如果我们需要在二次排序后保持原有排序的意义，就需要使用到稳定性的算法



## 插入排序

```go
func insertionSort(a []int, n int) {
	if n <= 1 {
		return
	}
	for i := 1; i < n; i++ {
		value := a[i]
		j := i - 1
		for ; j >= 0; j-- {
			if a[j] > value {
				a[j+1] = a[j]
			} else {
				break
			}
		}
		a[j+1] = value
	}
}
```



| Algo     | best    | worst   | average | in-place           | stable             |
| -------- | ------- | ------- | ------- | ------------------ | ------------------ |
| 插入排序 | O(n)    | O(n^2^) | O(n^2^) | :white_check_mark: | :white_check_mark: |
| 冒泡排序 | O(n^2^) | O(n^2^) | O(n^2^) | :white_check_mark: | :white_check_mark: |
| 选择排序 | O(n^2^) | O(n^2^) | O(n^2^) | :white_check_mark: | :x:                |

