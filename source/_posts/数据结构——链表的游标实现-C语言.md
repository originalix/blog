---
title: 数据结构——链表的游标实现(C语言)
date: 2017-07-20 08:14:12
tags: ["数据结构", "链表"]

---

上一篇博文我们用指针实现了链表，但是诸如BASIC和FORTRAN等许多语言都不支持指针。如果需要链表而又不能使用指针，这时我们可以使用游标（cursor）实现法来实现链表。

在链表的实现中有两个重要的特点：

- 数据存储在一组结构体中。每一个结构体包含有数据以及指向下一个结构体的指针。

- 一个新的结构体可以通过调用malloc而从系统全局内存（global memory）得到，并可以通过free而被释放。

游标法必须能够模仿实现这两条特性 。 下面给出实现代码:

<!--more-->

```c
#ifndef _CursorList_H

typedef int PtrToNode;
typedef PtrToNode List;
typedef PtrToNode Position;
typedef int ElementType;

void InitializeCursorSpace( void );

List MakeEmpty( List L );
int IsEmpty( const List L );
int IsLast( const Position P, const List L );
Position Find( ElementType X, const List L );
void Delete( ElementType X, List L );
Position FindPrevious( ElementType X, const List L);
void Insert( ElementType X, List L, Position P );
void DeleteList( const List L );
Position Header( const List L );
Position First( const List L);
Position Advance( const Position P );
ElementType Retrieve( const Position P );

#endif /*_CUrsor_H */
```
可以从上面的代码上看到，链表的游标实现跟链表的接口定义几乎是一样的。

下面放上实现代码:

```c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "CursorList.h"

#define OK 1
#define ERROR 0
#define TRUE 1
#define FALSE 0

#define SpaceSize 10

typedef int Status;

struct Node
{
    ElementType Element;
    Position Next;
};

struct Node CursorSpace[ SpaceSize ];

/* initialize the CursorSpace */
void InitCursorSpace()
{
    int i;
    for(i = 0; i < SpaceSize; i++)
        CursorSpace[i].Next = i == SpaceSize-1 ? 0 : i+1;
}

static Position CursorAlloc(void)
{
    Position P;

    P = CursorSpace[0].Next;
    CursorSpace[0].Next = CursorSpace[P].Next;

    return P;
}

static void CursorFree(Position P)
{
    CursorSpace[P].Next = CursorSpace[0].Next;
    CursorSpace[0].Next = P;
}

/* Return true if L is empty */
Status IsEmpty(List L)
{
    if (CursorSpace[L].Next == 0)
        return TRUE;
    else
        return FALSE;
}

/* Return true if P is the last position in list L */
/* Parameter L is unused in this implementation */
Status IsLast(Position P, List L)
{
    if (CursorSpace[P].Next == 0)
        return TRUE;
    else
        return FALSE;
}

/* Return Position of X in L; 0 if not found */
/* Uses a header node */
Position Find(ElementType X, List L)
{
    Position P;

    P = CursorSpace[L].Next;
    while(P && CursorSpace[P].Element != X) {
        P = CursorSpace[P].Next;
    }
    return P;
}

/* Delete first occurrence of X from a list */
/* Assume use of a header node */
void Delete(ElementType X, List L)
{
    Position P, TmpCell;

    P = FindPrevious(X, L);
    if (!IsLast(P, L))
        TmpCell = CursorSpace[P].Next;
        CursorSpace[P].Next = CursorSpace[TmpCell].Next;
        CursorFree(TmpCell);
}

/* Find the front of the first X of The list */
/* Return 0 if not found */
Position FindPrevious(ElementType X, const List L)
{
    Position P;
    P = L;
    while(P && CursorSpace[CursorSpace[P].Next].Element != X)
    {
        P = CursorSpace[P].Next;
    }
    return P;
}

/* Insert(after legal position P) */
/* Header implementation assumed */
/* Parameter L is unused in this implemention */
void Insert(ElementType X, List L, Position P)
{
    Position TmpCell;

    TmpCell = CursorAlloc();
    if (TmpCell == 0)
        printf("Out of space!\n");
    CursorSpace[TmpCell].Element = X;
    CursorSpace[TmpCell].Next = CursorSpace[P].Next;
    CursorSpace[P].Next = TmpCell;
}

void print_list(List L)
{
    List p = L;
    if (NULL == p)
    {
        printf("print_list: 链表为空!\n");
        return;
    }

    Position P;

    P = CursorSpace[L].Next;
    while(P != NULL) {
        printf("%d, \n", CursorSpace[P].Element);
        P = CursorSpace[P].Next;
    }
    printf("\n");
    return;
}

int main(int argc, char const *argv[])
{
    InitCursorSpace();
    List L = CursorAlloc();
    CursorSpace[L].Next = NULL;
    Insert(1, L, L);
    Insert(0, L, L);
    Insert(21, L, L);
    Insert(1201, L, L);
    Position P;
    P = Find(21, L);
    if (P)
        printf("找到元素: %d\n", CursorSpace[P].Element);
    else
        printf("未找到21元素\n");
    Delete(0, L);
    Delete(1, L);
    print_list(L);
    printf("检查链表是否为空: %d\n", IsEmpty(L));
    printf("Hello World\n");
    return 0;
}

```

实现过程比较简单，最后的main函数是对游标链表的测试。代码直接开箱即用，可以看到测试过程。