## 03-树2 List Leaves

Given a tree, you are supposed to list all the leaves in the order of top down, and left to right.

### Input Specification:

Each input file contains one test case. For each case, the first line gives a positive integer N (≤10) which is the total number of nodes in the tree -- and hence the nodes are numbered from 0 to N−1. Then N lines follow, each corresponds to a node, and gives the indices of the left and right children of the node. If the child does not exist, a "-" will be put at the position. Any pair of children are separated by a space.

### Output Specification:

For each test case, print in one line all the leaves' indices in the order of top down, and left to right. There must be exactly one space between any adjacent numbers, and no extra space at the end of the line.

Sample Input:

~~~
8
1 -
- -
0 -
2 7
- -
- -
5 -
4 6
~~~

Sample Output:

~~~
4 1 5
~~~

~~~ C

#include <stdbool.h>
#include <stdlib.h>
#include <stdio.h>

#define QUEUE_SIZE 16

struct tree {
    bool is_root;
    struct tree *left;
    struct tree *right;
};

struct queue {
    struct tree *queue[QUEUE_SIZE];
    int front;
    int rear;
};

struct tree *list = NULL;

int ctoi(char c)
{
    return c - '0';
}

struct tree *create_tree(void)
{
    int num;
    char left, right;
    int l, r;

    scanf("%d", &num);

    list = malloc(sizeof(struct tree) * num);
    for (int i = 0; i < num; i++) {
        list[i].is_root = true;
    }

    for (int i = 0; i < num; i++) {
        scanf(" %c %c", &left, &right);

        if (left != '-') {
            l = ctoi(left);
            list[i].left = &list[l];
            list[l].is_root = false;

        } else {
            list[i].left = NULL;
        }
    
        if (right != '-') {
            r = ctoi(right);
            list[i].right = &list[r];
            list[r].is_root = false;

        } else {
            list[i].right = NULL;
        }
    }

    for (int i = 0; i < num; i++) {
        if (list[i].is_root)
            return &list[i];
    }

    return NULL;
}

int enqueue(struct queue *q, struct tree *node)
{
    if (q->rear == (q->front - 1)) {
        return -1;
    }

    q->queue[q->rear] = node;
    q->rear = (q->rear + 1) % QUEUE_SIZE;
    return 0;
}

struct tree *dequeue(struct queue *q)
{
    struct tree *node;

    if (q->front == q->rear) {
        return NULL;
    }

    node = q->queue[q->front];
    q->front = (q->front + 1) % QUEUE_SIZE;
    return node;
}

bool queue_is_empty(struct queue *q)
{
    return q->front == q->rear;
}

int queue_size(struct queue *q)
{
    return q->rear >= q->front ?
           q->rear - q->front : q->rear + QUEUE_SIZE - q->front;
}

int level_order(struct tree *root)
{
    int n;
    struct tree *node;
    struct queue q = { .front = 0, .rear = 0 };
    struct queue leaves = { .front = 0, .rear = 0 };

    enqueue(&q, root);
    while (!queue_is_empty(&q)) {

        n = queue_size(&q);
        for (int i = 0; i < n; i++) {
            node = dequeue(&q);

            if (!node->left && !node->right) {
                enqueue(&leaves, node);

            } else {
                if (node->left)
                    enqueue(&q, node->left);
                if (node->right)
                    enqueue(&q, node->right);
            }
        }
    }

    n = queue_size(&leaves);
    while (queue_size(&leaves) > 1) {
        printf("%ld ", dequeue(&leaves) - list);
    }
    if (!queue_is_empty(&leaves))
        printf("%ld\n", dequeue(&leaves) - list);

    return 0;
}

int main(void)
{
    return level_order(create_tree());
}
~~~
