

# Golang标准库学习2-Sort

> sort包提供了对slice的排序以及用户自定义排序的function

#### 1、sort包的接口：

###### 1.1 Interface接口

```go
type Interface interface {
   Len() int										//elems的长度
   Less(i, j int) bool					//定义排序方式，若i<j则为升序排列
   Swap(i, j int)								//elems的swap
}
```

###### 1.2 reverse接口（对外不可见，等价于Interface接口就完事儿了）

```go
type reverse struct {
   Interface
}
func (r reverse) Less(i, j int) bool //重写了Interface里的less，实际上就是掉了个个
```

##### 2、Sort包的Func

###### 2.1 内部一些较为重要的接口

```go
//区间排序 -->Bubble -->sorts data[a:b]
func insertionSort(data Interface, a, b int)
//区间排序 --> 堆排序
func heapSort(data Interface, a, b int)
//区间排序 --> 快排 本质使用 小数组希尔排序+递归深度为0时堆排+三向切分快排策略（重复数据多的元素）
func quickSort(data Interface, a, b, maxDepth int) 

```

###### 2.2 暴漏给Coder的Func

```go
//排序 -->快排 --> 本质调用quickSort(data, 0, n, maxDepth(n))
func Sort(data Interface)
//获得一个reverse的Interface（注意，该方法不是直接把slice反转）
func Reverse(data Interface) Interface
//检查是否处于排序阶段
func IsSorted(data Interface) bool
//在 [0, n) 范围内搜索使得f为true的第一个索引，若返回n则代表均无符合条件的索引
func Search(n int, f func(int) bool) int
//稳定排序（相同元素位置不变）
func Stable(data Interface)
//几种直接传入slice并排序的func
func Ints(x []int) { Sort(IntSlice(x)) }
func Float64s(x []float64) { Sort(Float64Slice(x)) }
func Strings(x []string) { Sort(StringSlice(x)) }
//几种直接传入slice并查询是否排序的func
func IntsAreSorted(x []int) bool { return IsSorted(IntSlice(x)) }
func Float64sAreSorted(x []float64) bool { return IsSorted(Float64Slice(x)) }
func StringsAreSorted(x []string) bool { return IsSorted(StringSlice(x)) }
```

###### 2.3 暴漏给coder的几种实现了Interface的type

```go
//这些type均直接实现了Interface的三个方法
type StringSlice []string
type Float64Slice []float64
type IntSlice []int
```

##### 3、Sort包的实践

###### 3.1 一个int64的slice的排序实践

```GO
package main

import (
   "fmt"
   "sort"
   "strconv"
)

func main() {
   //Plan1 直接用暴漏出来的给slice用的接口
   var intSlice1 []int
   intSlice1 = []int{9, 44, 21, 234, 2, 12, 7, 879, 556, 4353, 36}
   sort.Ints(intSlice1)
   fmt.Printf("%#v\n", intSlice1)
   //Plan2 用sort包提供的已经实现接口的slice
   var intSlice2 sort.IntSlice
   intSlice2 = []int{9, 44, 21, 234, 2, 12, 7, 879, 556, 4353, 36}
   fmt.Printf("%#v\n", intSlice2)
   fmt.Printf("%#v\n", strconv.FormatBool(sort.IsSorted(intSlice2)))
   sort.Sort(intSlice2)//排序
   fmt.Printf("%#v\n", intSlice2)
   fmt.Printf("%#v\n", strconv.FormatBool(sort.IsSorted(intSlice2)))
   //返回第一个大于等于 30 的索引
   fmt.Printf("%#v\n", sort.Search(len(intSlice2), func(i int) bool {
      return intSlice2[i] >= 30}))
}

/*output
[]int{2, 7, 9, 12, 21, 36, 44, 234, 556, 879, 4353}
sort.IntSlice{9, 44, 21, 234, 2, 12, 7, 879, 556, 4353, 36}
"false"
sort.IntSlice{2, 7, 9, 12, 21, 36, 44, 234, 556, 879, 4353}
"true"
5
*/
```

###### 3.2 一个struct的排序实践

```go
package main
import (
   "fmt"
   "sort"
   "strconv"
)
type student struct {
   name      string
   mark      int
   studentID int64
}
type studentList []student
func (x studentList) Len() int {
   return len(x)
}
func (x studentList) Less(i, j int) bool {
   return x[i].mark > x[j].mark
}
func (x studentList) Swap(i, j int) {
   x[i], x[j] = x[j], x[i]
}
func main() {
   student1 := student{
      name:      "张三",
      mark:      64,
      studentID: int64(2019220331),
   }
   student2 := student{
      name:      "李四",
      mark:      32,
      studentID: int64(2019220212),
   }
   student3 := student{
      name:      "王二麻子",
      mark:      84,
      studentID: int64(2019220003),
   }
   stuList := studentList{student1, student2, student3}
   fmt.Println("--")
   fmt.Printf("%v\n", stuList)
   fmt.Printf("%v\n", strconv.FormatBool(sort.IsSorted(stuList)))
   sort.Sort(stuList) //对学生依据分数进行降序排序
   fmt.Printf("%v\n", stuList)
   fmt.Printf("%v\n", strconv.FormatBool(sort.IsSorted(stuList)))
}

/*
输出信息：
[{张三 64 2019220331} {李四 32 2019220212} {王二麻子 84 2019220003}]
false
[{王二麻子 84 2019220003} {张三 64 2019220331} {李四 32 2019220212}]
true
*/
```

