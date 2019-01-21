---
title: 数据结构——AVL树(C语言)
date: 2017-08-06 09:30:12
tags: ["数据结构", "AVL树"]

---

AVL(Adelson-Velskii 和 Landis)树是带有平衡条件的二叉查找树。在计算机科学中，AVL树是最先发明的自平衡二叉查找树。在AVL树中任何节点的两个子树的高度最大差别为1，所以它也被称为高度平衡树。查找、插入和删除在平均和最坏情况下的时间复杂度都是O(lngn)。增加和删除可能需要通过一次或多次树旋转来重新平衡这个树。

节点的平衡因子是它的左子树的高度减去它的右子树的高度（有时相反）。带有平衡因子1、0或-1的结点被认为是平衡的。带有平衡因子-2或2的节点被认为是不平衡的，并需要重新平衡这个树。平衡因子可以直接存储在每个节点中，或从可能存储在节点中的子树高度计算出来。

<!--more-->

AVL树的基本操作一般涉及运作同在不平衡的二叉查找树所运作的同样的算法。但是要进行预先或随后做一次或多次所谓的"AVL旋转"。

以下图标表示的四种情况，就是AVL旋转中常见的四种。（图片用了维基百科的，不确定不开vpn图是否会挂）。

![](https://upload.wikimedia.org/wikipedia/commons/c/c7/Tree_Rebalancing.png)

下面来看AVL树的操作有哪些:

```c
#ifndef _AvlTree_H

struct AvlNode;
typedef struct AvlNode *Position;
typedef struct AvlNode *AvlTree;
typedef int ElementType;

AvlTree MakeEmpty( AvlTree T );
Position Find( ElementType X, AvlTree T );
Position FindMin( AvlTree T );
Position FindMax( AvlTree T );
AvlTree Insert( ElementType X, AvlTree T );
AvlTree Delete( ElementType X, AvlTree T );
ElementType Retrieve( Position P );

#endif /* _AvlTree_H */
```

下面是对于上面操作定义的实现:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "AvlTree.h"

#define OK 1
#define ERROR 0
#define TRUE 1
#define FALSE 0

typedef int Status;

struct AvlNode
{
    ElementType Element;
    AvlTree Left;
    AvlTree Right;
    int Height;
};

AvlTree MakeEmpty(AvlTree T)
{
    if (T != NULL)
    {
        MakeEmpty(T->Left);
        MakeEmpty(T->Right);
        free(T);
    }
    return NULL;
}

/**
 * 计算Avl节点高度
 * @param  P 节点P
 * @return 树高
 */
static int Height(Position P)
{
    if (P == NULL)
        return -1;
    else
        return P->Height;
}

static int Max(int a, int b)
{
    return a > b ? a : b;
}

/* 向左单旋 */
static Position SingleRotateWithLeft(Position K2)
{
    Position K1;

    K1 = K2->Left;
    K2->Left = K1->Right;
    K1->Right = K2;

    K2->Height = Max(Height(K2->Left), Height(K2->Right)) + 1;
    K1->Height = Max(Height(K1->Left), K2->Height) + 1;

    return K1; /* New Root */
}

/* 向右单旋  */
static Position SingleRotateWithRight(Position K2)
{
    Position K1;

    K1 = K2->Right;
    K2->Right = K1->Left;
    K1->Left = K2;

    K2->Height = Max(Height(K2->Left), Height(K2->Right)) + 1;
    K1->Height = Max(K2->Height, Height(K1->Right)) + 1;

    return K1; /*New root */
}

/* 向左双旋 */
static Position DoubleRotateWithLeft(Position K3)
{
    /* Rotate between K1 and K2 */
    K3->Left = SingleRotateWithRight(K3->Left);

    /* Rotate between K3 and K2 */
    return SingleRotateWithLeft(K3);
}

/* 向右双旋 */
static Position DoubleRotateWithRight(Position K3)
{
    K3->Right = SingleRotateWithLeft(K3->Right);

    return SingleRotateWithRight(K3);
}

AvlTree Insert(ElementType X, AvlTree T)
{
    if (T == NULL)
    {
        T = malloc( sizeof( struct AvlNode ) );
        if (T == NULL)
            printf("Out of space!!!\n");
        else
        {
            T->Element = X;
            T->Height = 0;
            T->Left = T->Right = NULL;
        }
    }
    else if (X < T->Element) /* 左子树插入新节点 */
    {
        T->Left = Insert(X, T->Left);
        if (Height(T->Left) - Height(T->Right) == 2)
            if (X < T->Left->Element)
                T = SingleRotateWithLeft(T);
            else
                T = DoubleRotateWithLeft(T);
    }
    else if (X > T->Element) /* 右子树插入新节点 */
    {
        T->Right = Insert(X, T->Right);
        if (Height(T->Right) - Height(T->Left) == 2)
            if (X > T->Right->Element)
                T = SingleRotateWithRight(T);
            else
                T = DoubleRotateWithRight(T);
    }
    /* Else X is in the tree alredy; we'll do nothing */
    T->Height = Max(Height(T->Left), Height(T->Right)) + 1;
    return T;
}

AvlTree Delete(ElementType X, AvlTree T)
{
    Position TmpCell;
     if(T == NULL) {
        printf("没找到该元素，无法删除！\n");
        return NULL;
     }
     else if (X < T->Element)
         T->Left = Delete(X, T->Left);
     else if (X > T->Element)
         T->Right = Delete(X, T->Right);
     else if(T->Left && T->Right) { //要删除的树左右都有儿子
         TmpCell = FindMin(T->Right);   //用该结点右儿子上最小结点替换该结点，然后与只有一个儿子的操作方法相同
         T->Element = TmpCell->Element;
         T->Right = Delete(T->Element, T->Right);
     }else{
         TmpCell = T;        //要删除的结点只有一个儿子
         if(T->Left == NULL)
             T = T->Right;
         else if(T->Right == NULL)
             T = T->Left;
         free(TmpCell);
     }
     return T;
}

/* 查找X元素所在的位置 */
Position Find(ElementType X, AvlTree T)
{
    if (T == NULL)
        return NULL;
    if (X < T->Element)
        return Find(X, T->Left);
    else if (X > T->Element)
        return Find(X, T->Right);
    else
        return T;
}

/* search the min element in AvlTree*/
Position FindMin(AvlTree T)
{
    if (T == NULL)
        return NULL;
    else if (T->Left == NULL)
        return T;
    else
        return FindMin(T->Left);
}

/* search the max element in AvlTree */
Position FindMax(AvlTree T)
{
    if (T == NULL)
        return NULL;
    else if (T->Right == NULL)
        return T;
    else
        return FindMax(T->Right);
}

ElementType Retrieve(Position P)
{
    if(P != NULL)
        return P->Element;
    return -1;
}

/**
 * 前序遍历"二叉树"
 * @param T Tree
 */
void PreorderTravel(AvlTree T)
{
    if (T != NULL)
    {
        printf("%d\n", T->Element);
        PreorderTravel(T->Left);
        PreorderTravel(T->Right);
    }
}

/**
 * 中序遍历"二叉树"
 * @param T Tree
 */
void InorderTravel(AvlTree T)
{
    if (T != NULL)
    {
        InorderTravel(T->Left);
        printf("%d\n", T->Element);
        InorderTravel(T->Right);
    }
}

/**
 * 后序遍历二叉树
 * @param T Tree
 */
void PostorderTravel(AvlTree T)
{
    if (T != NULL)
    {
        PostorderTravel(T->Left);
        PostorderTravel(T->Right);
        printf("%d\n", T->Element);
    }
}

/* 打印二叉树信息 */
void PrintTree(AvlTree T, ElementType Element, int direction)
{
    if (T != NULL)
    {
        if (direction == 0)
            printf("%2d is root\n", T->Element);
        else
            printf("%2d is %2d's %6s child\n", T->Element, Element, direction == 1 ? "right" : "left");

        PrintTree(T->Left, T->Element, -1);
        PrintTree(T->Right, T->Element, 1);
    }
}
```

在实现完成这些函数后，我们在`main`函数中对AVL树进行测试:

```c
int main(int argc, char const *argv[])
{
    printf("Hello World\n");

    AvlTree T;
    Position P;
    int i;

    T = MakeEmpty(NULL);

    T = Insert(21, T);
    T = Insert(2150, T);
    T = Insert(50, T);
    T = Insert(12, T);
    T = Insert(1201, T);

    printf("Root: %d\n", T->Element);

    printf("树的详细信息: \n");
    PrintTree(T, T->Element, 0);

    printf("前序遍历二叉树: \n");
    PreorderTravel(T);

    printf("中序遍历二叉树: \n");
    InorderTravel(T);

    printf("后序遍历二叉树: \n");
    PostorderTravel(T);

    printf("最大值: %d\n", FindMax(T)->Element);
    printf("最小值: %d\n", FindMin(T)->Element);

    Delete(50, T);
    printf("树的详细信息: \n");
    PrintTree(T, T->Element, 0);

    return 0;
}

```
