## 01-复杂度3 二分查找

本题要求实现二分查找算法。

### 函数接口定义：

Position BinarySearch( List L, ElementType X );

L是用户传入的一个线性表，其中ElementType元素可以通过>、==、<进行比较，并且题目保证传入的数据是递增有序的。函数BinarySearch要查找X在Data中的位置，即数组下标（注意：元素从下标1开始存储）。找到则返回下标，否则返回一个特殊的失败标记NotFound。

### 输入样例1：

~~~
5
12 31 55 89 101
31
~~~

### 输出样例1：

~~~
2
~~~

~~~ c
#include <stdio.h>
#include <stdlib.h>

#define MAXSIZE 10
#define NotFound 0
typedef int ElementType;

typedef int Position;
typedef struct LNode *List;
struct LNode {
    ElementType Data[MAXSIZE];
    Position Last; /* 保存线性表中最后一个元素的位置 */
};

List ReadInput(); /* 裁判实现，细节不表。元素从下标1开始存储 */
Position BinarySearch( List L, ElementType X );

int main(void)
{
    List L;
    ElementType X;
    Position P;

    L = malloc(sizeof(struct LNode));
    int l[] = {12,31,55,89,101,120};
    L->Last = 6;
    for (int i = 1; i <= L->Last; i++) {
        L->Data[i] = l[i - 1];
        printf("%d\n", L->Data[i]);
    }

    P = BinarySearch( L, 12);
    printf("%d\n", P);

    return 0;
}

Position BinarySearch( List L, ElementType X )
{
    int *l = L->Data;
    int index = NotFound;
    int mid, left = 1, right = L->Last;

    while (left <= right) {
        mid = (left + right) >> 1;
        if (l[mid] > X)
            right = mid - 1;
        else if (l[mid] < X)
            left = mid + 1;
        else {
            index = mid;
            break;
        }
    }
    return index;
}
~~~
