---
title: 创造一颗自己的二叉树
date: 2020-07-09 01:33:25
tags:
     - 学习
     - 数据结构
     - C
---

继链表后，二叉树成为了我学习数据结构道路上的第二只拦路虎

二叉树不仅仅复杂在每个节点有两个子节点，更复杂在其本身的操作上。

在链表中可以用循环搞定的东西，到了二叉树就全变成 `递归` 了

所以，二叉树可以说是前一章的结束，下一章的开始。

---

**前排提示，我不喜欢在我的代码中加入各式各样的注释**

**如果你能接受我的任性，请继续**

---

从二叉树还延伸出各种各样令人头疼的数据结构：

- AVL 树

- 自平衡二叉树

- 红黑树

- B 树

- B+树

……

二叉树的学习，是为了向下一个知识进军

---

## 设计一颗 `树根`

俗话说 `万事开头难` 只要抓住树根的本质，我们就能更随心所欲地操纵二叉树

以下给出 `C 语言` 的实现方式

值得一提的是，在这篇文章的大部分环节，我都会使用 `C 语言`

因为我想通过 C 对数据结构的学习，加深我对底层操作的理解

```C
typedef struct treeNode {
    struct treeNode *lChild;
    struct treeNode *rChild;
    int data;
} treeNode;
```

以上代码实现了一个树节点的定义

学过 `链表` 的同学肯定不会陌生，当创建一个 `链表节点` 时

我们必须要让这个节点具有指向下一个节点的能力

我们可以将之理解为 `结点指针`

让我们把这个代码拆成两部分

```C
typedef struct treeNode {...} treeNode;
```

```C
struct treeNode {
    struct treeNode *lChild;
    struct treeNode *rChild;
    int data;
}
```

怎么样，是不是感觉明白了一点

其实 `typedef` 的作用就是方便我们调用这个节点的类型

---

## 种植 `树根`

光有了一个节点怎么足够呢，我们要让这棵树长起来，一个个结点的 `创建` ， `添加` 便是不可缺少之事

但是我们不能放任树根肆意生长，我们要保证树枝在产生时，一定是

- 稳定的（有分配足够的内存）
- 可持续的（方便后代 `嫁接` 到这个根的末尾）

所以，我们要自己定义函数，来规范树的生长

```C
treeNode *buildTree(int rootData) {
    treeNode *root = malloc(sizeof(treeNode));
    root->data = rootData;
    root->lChild = NULL;
    root->rChild = NULL;

    return root;
}
```

在这里， `malloc` 函数给予树根足够的空间生长

而 `NULL` 值的设定可以保证对于树枝的访问是安全的，用户不会读到一大堆莫名其妙的数据

考虑更周到的人还可以思索一下当 `malloc` 失败时：

- 作出什么样的提示是符合用户想法的

- 能否允许用户进行一些他们自己心里有数的过界操作

---

## 让树 `生长`

我们已经种植下去了一个树根，接下来就要让他成长

毕竟，只有一棵树根的树除了能在内存洪流中绊人一脚之外，也没什么用处

```C
void addNode(treeNode *root, int nodeData) {
    if (nodeData == root->data) {
        printf("repeat");
        return;
    }

    if (nodeData > root->data) {
        if (root->rChild == NULL) {
            root->rChild = buildTree(nodeData);
        } else {
            addNode(root->rChild, nodeData);
        }
    } else {
        if (root->lChild == NULL) {
            root->lChild = buildTree(nodeData);
        } else {
            addNode(root->lChild, nodeData);
        }
    }
}
```

在这里，我们就要正式面对那些令人胆寒的递归操作了

大部分初步接受计算机的人基本都会认为递归操作很神奇，认为它在冥冥之中有一丝简洁美

但是用**自己定义自己**这一行为又不会让人觉得踏实

```C
if (root->rChild == NULL) {
    root->rChild = buildTree(nodeData);
} else {
    addNode(root->rChild, nodeData);
}
```

我们将它的一部分代码拿出来仔细观察，而也只有在这时，我们才会体会到递归的美丽和优雅

我们可以看到先前 `buildTree` 时留下的 `NULL` 树根帮了大忙，它帮我们很好地区分了已经使用的节点和未曾使用的节点

当子节点未使用且符合条件时，我们把数值连接到对应的子节点上，并令这个节点也充分舒展（ `buildTree` 将发挥它伟大的作用 ）

当**子节点**不满足条件时，我们就把**子节点的子节点**也传递进去，如果**子节点的子节点**也不满足，他会接着向**其子节点**传递……

最终找到那个满足条件的树根，函数之后就**一层层地将那个树根返回**

---

在顿悟递归的一瞬间，你就会被它简洁优美的内核震撼

这在我们脑中理所当然的思想，似乎从来没有被开发过

---

## 我的树长了 `多高` 了

```C
int height(treeNode *root) {
    if (!root) { return 0; }
    if (root->rChild == NULL && root->lChild == NULL) { return 1; }
    int leftHeight = 0;
    int rightHeight = 0;

    if (root->lChild) {
        leftHeight += 1;
        leftHeight += height(root->lChild);
    }

    if (root->rChild) {
        rightHeight += 1;
        rightHeight += height(root->rChild);
    }

    return (leftHeight > rightHeight) ? leftHeight : rightHeight;
}
```

从这串代码里，我们能看出递归对于树的重要性

一般，评价树的高度是找最长的那一根分支

当我们从上往下找时，最长的那一根分支可能左拐，右拐，中间截断，人类肉眼尚且不能一步观测到最长的分支

而递归用了一个笨方法，它先把自己的计数设定为零，随后找到所有节点的最根部

一个一个地向上返回，而且只返回最大值，一个新生的节点肯定比不过从更古老节点传上来的长度，正如

```C
return (leftHeight > rightHeight) ? leftHeight : rightHeight;
```

---

## 那个苹果 `在哪里`

```C
treeNode *findNode(treeNode *root, int findData) {
    if (root) {
        if (root->data == findData) {
            return root;
        } else if (root->data > findData) {
            findNode(root->lChild, findData);
        } else {
            findNode(root->rChild, findData);
        }
    }
    return NULL;
}
```

你可以将其理解为 `addNode` 的简化版，只有比较和输出，并没有添加，删除，修改操作

当目标值较大时朝右边走，较小时朝左边走，是一个可能被人忽视的小技巧

但是不要过于堕落，你甚至只需要走半步 —— 把参数传进去，剩下的东西让递归帮你完成

---

## 亲爱的小姐，能请您帮我 `描述一下` 这棵树吗

```C
void frontErgodic(treeNode *root) {
    if (root) {
        printf("%d\n", root->data);
        frontErgodic(root->lChild);
        frontErgodic(root->rChild);
    }
}

void middleErgodic(treeNode *root) {
    if (root) {
        middleErgodic(root->lChild);
        printf("%d\n", root->data);
        middleErgodic(root->rChild);
    }
}

void backErgodic(treeNode *root) {
    if (root) {
        backErgodic(root->lChild);
        backErgodic(root->rChild);
        printf("%d\n", root->data);
    }
}
```

我们一般把上面三种方式分别称为： `前序遍历` 、 `中序遍历` 和 `后序遍历`

仔细观察 `printf` 的位置，再加上你对递归的一点点理解，看懂这一步并不困难

我总是不愿意使用这类功能，因为我常常直接在 `debug` 时直接观察它的结构

---

## 我需要把这颗长坏的树枝 `锯下来`

```C
void deleteNode(treeNode *root, int deleteData) {
    if (root) {
        treeNode *LCFind = findNode(root->lChild, deleteData);
        treeNode *RCFind = findNode(root->rChild, deleteData);
        if (LCFind == NULL && RCFind == NULL) {
            printf("delete failed, data %d not exist\n", deleteData);
            return;
        }
        treeNode *tmp = NULL;

        if (LCFind) {
            if (LCFind->rChild == NULL && LCFind->lChild == NULL) {
                free(LCFind);
                root->lChild = NULL;
            } else if (LCFind->lChild && LCFind->rChild) {
                tmp = LCFind->lChild;
                linkNode(tmp, LCFind->rChild);
                free(LCFind);
                root->lChild = tmp;
            } else if (LCFind->lChild) {
                tmp = LCFind->lChild;
                free(LCFind);
                root->lChild = tmp;
            } else {
                tmp = LCFind->rChild;
                free(LCFind);
                root->lChild = tmp;
            }

        } else {
            if (RCFind->rChild == NULL && RCFind->lChild == NULL) {
                free(LCFind);
                root->rChild = NULL;
            } else if (RCFind->lChild && RCFind->rChild) {
                tmp = RCFind->lChild;
                linkNode(tmp, RCFind->rChild);
                free(RCFind);
                root->rChild = tmp;
            } else if (RCFind->lChild) {
                tmp = RCFind->lChild;
                free(RCFind);
                root->rChild = tmp;
            } else {
                tmp = RCFind->rChild;
                free(RCFind);
                root->lChild = tmp;
            }
        }
    }
}
```

看看这复杂的代码，不得不承认，删除二叉树中的某一个节点真的是一件费力的事情

对于每一个节点，我们需要知道：

- 如果它是要被删除的节点：

  - 他的子节点有多少个？（ 0、1、2 ）
  - 节点 `free` 后，如何处理**父节点**
  - 我如何处理才能将原来的数据有序排放

- 如果他不是要被删除的节点：

  - 这是最后一个节点了吗
  - 下一个节点是什么
  - 我该往哪里走

递归查找部分，早在代码的开始已经完成，即：

```C
treeNode *LCFind = findNode(root->lChild, deleteData);
treeNode *RCFind = findNode(root->rChild, deleteData);

if (LCFind == NULL && RCFind == NULL) {
    printf("delete failed, data %d not exist\n", deleteData);
    return;
}
```

单单是一次情况判断，就足够复杂

---

### 为什么定义 `LCFind` 和 `RCFind`

（ 解释：`left-child-find` 和 `right-child-find` ）

因为我们删除后，一定要能找到他的**父节点**，并且把不需要删除的节点**重新接回去**

`LCFind` 和 `RCFind` 是 `root` 的左右子节点，自然不能代替掉 `root` 存在

---

### `linkNode` 是什么

```C
void linkNode(treeNode *left, treeNode *right) {
    if (left && right) {
        if (left->rChild == NULL) {
            left->rChild = right;
        } else {
            linkNode(left->rChild, right);
        }
    }
}
```

它是一台接骨机器，能将**将要被删除的节点**的**右边子节点集合**的 `root` 位准确地接到**左侧子节点集合**的最大位，保证树的有序

---

对于从未学过数据结构的我来说，二叉树着实为我打开了一扇天窗

让我能从简单的指针应用中跳脱出来，真正拿他做一些更灵活，更有趣的事情

之前我对指针的 `线性认知` 也更新到了 `平面认知` ，指针的应用变得更加灵活了

---

我唯一的遗憾，就是目前还没有发现消减分支的好办法

---

二叉树的构建是很简单的，只需要一个劲往下面数数，比较，然后添加节点

而 `AVL树` 、 `红黑树` 、 `B树` 的每一次添加，删除都有可能造成数的不稳定，从而导致树的结构发生翻天覆地的变化

在探索数据结构的道路上，我暂时还是不会停下来的

```C
int main() {
    printf("Hello, world");
}
```

你好，世界
