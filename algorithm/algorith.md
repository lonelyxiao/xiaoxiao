# 基础概念

- 什么是数据结构
  - 存储数据的不同方式
- 什么是算法
  - 同一个问题的不同解决方法

## 计算机基础

### 原码、反码、补码

- 机器数

  - 一个数在计算机中的二进制表示形式,  叫做这个数的机器数

  - 比如，十进制中的数 +3 ，计算机字长为8位，转换成二进制就是00000011。如果是 -3 ，就是 10000011 。

    那么，这里的 00000011 和 10000011 就是机器数

- 真值

  - 就是现实中的数字（二进制）必须有+/-，实际中整数舍弃了+。
  - 例：0000 0001的真值 = +000 0001 = +1，1000 0001的真值 = –000 0001 = –1

- 原码

  - 例如，我们用8位二进制表示一个数，+11的原码为00001011，-11的原码就是10001011
  - 在计算机中表示的带符号的二进制数称为机器数。原码是机器数的一种表示方式

```tex
真值：+10000101        -10101100
原码：010000101        110101100
```

- 反码

  -  符号位不动
  - 正数的反码是其本身
  - 负数的反码是其余各个位取反

```
[+1] = [00000001]原 = [00000001]反

[-1] = [10000001]原 = [11111110]反
```

- 补码
  - 正数的补码就是其本身
  - 负数的补码是在其原码的基础上, 符号位不变, 其余各位取反, 最后+1. (**即在反码的基础上+1**)

```tex
真值：+1010111          -1110101    -101010100
反码：01010111			 10001010	 1010101011
补码：01010111          10001011    1010101100
```



#  斐波那契数列

斐波那契数列（1,1,2,3,5,8,13,21,34 ），现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。n<=39

## 递归解法

从数值中可以看到，第3项的值=第2+第1项值，即 f(n) = f(n-1) + f(n-2)

其时间复杂度为：2^n

```java
public int fibonacci(int n) {
    if(n==0){
        return 0;
    }
    if(n == 1) {
        return 1;
    }
    if(n>1){
        int num = this.fibonacci(n-1) + this.fibonacci(n-2);
        return num;
    }
    return 0;
}
```

## 快速解法

n=2   res=1+0

n=3  res = (1+0)+1

n=4  res= (1+0) + (1+0+1)

n=5  res = (1+0+1) +( (1+0) + (1+0+1))

从上面看，n=5 时，（1+0+1）在n=3时计算过， ( (1+0) + (1+0+1)) 在n=4时计算过，所以，代码上，可以弄两个变量，保存第n-1项，和n-2项的值

```java
public int fibonacci(int n) {
    if(n==0){
        return 0;
    }
    if(n == 1) {
        return 1;
    }
    int a=0;
    int b=1;
    int ret=0;
    for (int i=1; i<n; i++){
        //保留前两项之和
        ret =a+b;
        //下一轮b=n-2项的值
        a=b;
        //下一轮ret=n-1项的值
        b=ret;
    }
    return ret;
}
```

## 青蛙跳台阶

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）

解法：

由于它可以跳1级台阶或者2级台阶，所以**它上一步必定在第n-1,或者第n-2级台阶**，也就是说**它跳上n级台阶的跳法数是跳上n-1和跳上n-2级台阶的跳法数之和**



```java
public class Solution {
    public int jumpFloor(int target) {
        if(target <=2)
            return target;
        int ret=0, a=1, b=2;
        for(int i=2; i<target; i++){
            ret=a+b;
            a=b;
            b=ret;
        }
        return ret;
    }
}
```

## 变态跳台阶

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级，求该青蛙跳上一个n级的台阶总共有多少种跳法。 f(n)=f(n-1)+...+f(1)， f(n-1) = f(n-2) + ... + f(1) f(n)=2f(n-1)

```java
public class Solution {
    public int jumpFloorII(int target) {
        if(target ==1)
            return 1;
        int ret =1;
        int a=1; //相当于第一步
        //循环n次
        for(int i=2; i<=target; i++){
            //当n=2时， 2*（n=1）
            ret = 2*a;
            a=ret;
        }
        return ret;
    }
}
```

#  位运算

##  二进制中1的个数

输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

```java
public int NumberOf1(int n) {
    int count=0;
    while (n!=0) {
        //当n&(n-1)相当于去掉了最低位的1
        //10010&10001=10000
        n=n&(n-1);
        count++;
    }
    return count;
}
```

## 不用加减乘除做加法

1.不考虑进位对每一位相加。0加0、1加1结果都是0,0加1、1加0结果都是1。这和异或运算一样；

2.考虑进位，0加0、0加1、1加0都不产生进位，只有1加1向前产生一个进位。可看成是先做位与运算，然后向左移动一位；

3.相加过程重复前两步，直到不产生进位为止。(如果有进位，则需要进位+原有位数的数)

如：

010+010

010^010 +010&010<<1 = 0000+1000

```java
static int add(int a, int b) {
    while (b!=0) {
        //异或计算
        int temp = a^b;
        //计算进位
        b = (a&b)<<1;
        a = temp;
    }
    return a;
}
```

## 数组中出现次数超过一半的数字

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

### 方式一

- 空间复杂度O(n), 时间复杂度O(n)

定义一个map，记录数字和次数<数字， 次数>，记录完后遍历输出值

### 方式二

- 数组中的数两两兑换，如果不相等，则去掉，留下来的数则可能是满足条件的数
- 如： 1 2 不相等，去掉， 3 2 去掉， 2 5 去掉， 4 2 去掉， 剩下的2 2 就是数组中最多的一个数，可能会超过1/2

```java
//记录当前比对的对数的前一个数
//如： 1 2  中的 1
int last = 0;
int lastCount = 0;
for(int num : array) {
    if(lastCount == 0) {
        //记录两个数中的前一个数
        last = num;
        lastCount = 1;
    } else {
        if(last == num) {
            lastCount++;
        } else {
            //当不相等时，则-1，一般两个不相等的，此时需要减到0
            lastCount--;
        }
    }
}
if(lastCount == 0) {
    return 0;
} else {
    int length = array.length;
    //循环查出出现的次数
    int sum = 0;
    for(int num : array) {
        if(num == last){
            sum ++;
        }
        if(sum>(length>>1)) {
            return last;
        }
    }
}
return 0;
```

# 树

## 二叉树

结构：

```java
static class Node<T> {
    T val;
    Node<T> left;
    Node<T> right;
    public Node(T val) {
        this.val = val;
    }
}
```

- 深度优先

![image-20210630210023215](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210630210023.png)

![image-20210630210907545](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210630210907.png)

1. 先序遍历：先打印根节点，再打印左子树，再打印右子树

```java
public static void firstOrder(Node<Integer> node) {
    if(node == null) {
        return;
    }
    System.out.print(node.val + ",");
    firstOrder(node.left);
    firstOrder(node.right);
}
```

1. 中序遍历：左子树输出完了后再输出根节点，再输出右子树

```java
public static void inorder(Node<Integer> node) {
    if(node == null) {
        return;
    }
    inorder(node.left);
    System.out.print(node.val + ",");
    inorder(node.right);
}
```

1. 后续遍历：先左后右再根

![image-20210630212525421](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210630212525.png)

- 广度优先

![image-20210630210928543](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210630210928.png)

## 二叉搜索树

- 插入时，数据比当前节点小放在左子树，比当前节点大放在右子树

## 2-3数



# LRU

1. 某个key的set或get操作一旦发生，认为这个key的记录成了最常使用的。
2. 当缓存的大小超过K时，移除最不经常使用的记录，即set或get最久远的

特性：

- 必须要有顺序之分，以区分最近使用的和很久没有使用的数据排序
- 写和读操作一次搞定
- 如果容量(坑位)满了要删除最不长用的数据，每次新访问还要把新的数据插入到队头(按照业务你自己设定左右那一边是队头)

所以采用：hashmap+双向链表来进行设计

![image-20210630200312887](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210630200320.png)