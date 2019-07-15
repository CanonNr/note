# JAVA中各种数组、集合、列表区别



## Array数组

### 特点：

- 需声明长度
- 内存中连续存储
- 可以重复

### 优劣

* 优势
  * 索引速度是非常的快，而且赋值与修改元素也很简单
* 劣势
  * 声明时必须同时声明数组长度，在不知道数组长度时比较麻烦

### 代码示例：

```java
// 声明
string[] array = new string[3];
// 赋值
array[0]="孙悟空"; 
array[1]="猪八戒"; 
array[2]="沙悟净";
// 修改
array[1]="b1";
// 输出
System.out.print(array[1]);
```



## ArrayList

### 特点：

* 是动态数组实现的列表
* 有序
* 可以重复
* 数组特性

### 优劣：

* 优势
  * 无需声明长度，可以自动扩容。

  * ArrayList继承了List接口，有着比较方便的操作。

  * 可以插入多种不同类型的数据。

  * 根据下标遍历元素效率较高。

    

* 劣势
  * 插入和删除的效率比较低。
  * 根据内容查找元素的效率较低。

  

### 代码展示：

```java
// 声明
ArrayList arraylist = new ArrayList();
// 添加
arraylist.Add("孙悟空");
arraylist.Add(10086);
arraylist.Add(new Student());
// 修改
arraylist[3] = 10010;
// 删除
arraylist.RemoveAt(0);
// 插入
arraylist.Insert(0, "hello world");
// 输出
System.out.print(arraylist.get());
```



## LinkedList

### 特点：

- 不可以重复
- 基于链表实现的list所以是链表特性
- 利于做队列

### 优劣：

* 优势
  * 添加删除值很快。
  * 无需声明长度，可以自动扩容。

  

* 劣势

  * 更消耗内存

### 代码展示：



## Set（集合）

### 特点：

- 不可以重复



