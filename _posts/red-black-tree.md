---
title: 红黑树
date: 2018-05-30 20:18:17
tags: [数据结构与算法, 红黑树]
categories: 数据结构与算法
---

# 特征

- 所有的节点必须是红节点或黑节点
- 根节点是黑色
- 每个空叶子节点（不同于二叉搜索树，红黑树的叶子节点均为空，即只有黑颜色属性，无其他属性）为黑色
- 红节点的子节点为黑色（即：不能连续两个父子节点均为红节点）
- 从任意节点到其叶子节点经过的路径的黑节点数相同，即拥有相同的黑路径

<!-- more -->

# 插入

首先根据二叉搜索树的插入方式找到要插入的位置，然后将新插入的节点标为红色。此处约定：要插入的节点为N，N的父节点为P，N的祖父节点（即P的父节点）为G，N的叔父节点（即P的兄弟节点）为U。



```flow
start=>start: Start
hasFather=>condition: 有无父节点P
isFatherNodeColorRed=>condition: 父节点P为红色
isUncleNodeColorBlack=>condition: 叔父节点U为黑色
isNewNodeInside=>condition: 节点N是否在树的内侧
						  （G[左] - > P[右] - > N
						  或G[右] - > P[左] - > N）
handleCase4_1=>operation: 对P进行左旋转或右旋转
						左旋：G[左] - > P[右] - > N
						右旋：G[右] - > P[左] - > N
						旋转完后，P在树的外侧，即
						G[左] - > N[左] - > P
						或G[右] - > N[右] - > P
						将P当成N，N当成P
handleCase4_2=>operation: 对G进行左旋转或右旋转
						右旋：G[左] - > P[左] - > N
						左旋：G[右] - > P[右] - > N
						将父节点、祖父节点颜色反转
						（即P改为黑色，G改为红色）
handleCase3=>operation: 将父节点、叔父节点、祖父节
					  点颜色反转（即P改为黑色，
					  U改成黑色，G改为红色）
					  将G当成新插入的节点N
handleCase1=>operation: 将节点N颜色变为黑色
handleCase2=>operation: 直接插入不用其他处理
end=>end: End

start->hasFather
hasFather(yes)->isFatherNodeColorRed
isFatherNodeColorRed(yes)->isUncleNodeColorBlack
isUncleNodeColorBlack(no)->handleCase3->hasFather
isUncleNodeColorBlack(yes)->isNewNodeInside
isNewNodeInside(yes)->handleCase4_1->handleCase4_2->end
isNewNodeInside(no)->handleCase4_2
hasFather(no)->handleCase1->end
isFatherNodeColorRed(no)->handleCase2->end

```

# 删除

类似二叉搜索树，先找到要删除的数所在的节点V，对以这个节点为根节点的左子树进行搜索找到左子树的最大值所在的节点N（此处均以N在V的左子树进行讨论），将N节点的值覆盖到V节点上，然后将N节点进行删除。根据二叉搜索树的定义，可知，N节点最多有一个非叶子节点，对于搜索二叉树而言，直接删除N节点，将N节点的父节点P指向N节点即可。对于红黑树而言，除了执行跟搜索二叉树一样的操作外，还需要进行额外的调整使得整颗树仍然符合红黑树的特征，以下是进行调整的过程，约定如下：

- N：要删除的节点
- C：要删除的节点的子节点（排除另一个必为叶子的子节点，此子节点可能为叶子节点也可能非叶子节点）
- P：要删除的节点的父节点
- S：要删除的节点的兄弟节点
- SL：要删除的节点的兄弟节点的左节点
- SR：要删除的节点的兄弟节点的右节点
- 每一轮循环都根据N重新计算C、P、S、SL、SR节点

```flow
start=>start: Start
isNodeBlack=>condition: N为黑色
isChildBlack=>condition: C为黑色
isChildLeaf=>condition: C为叶子节点
handleCase1=>operation: C颜色改为黑色
isNodeNotRoot=>condition: N非根节点
isSiblingNodeBlack=>condition: S节点为黑色
handleCase2=>operation: 反转P和S的颜色
					  （P为红，S为黑）,
					  左旋或右旋P
isSRNodeBlack=>condition: SR为黑色
handleCase3=>operation: 将P和S颜色对换，
					  SR设为黑色，旋转
					  P让S成为P的父节点
isSLNodeBlack=>condition: SL为黑色
handleCase4=>operation: 旋转S让SL成为S的父节点
isParentNodeRed=>condition: P节点为红色
handleCase5=>operation: P改成黑色，
					  S改成红色
handleCase6=>operation: S改成红色，
				       将P当成N
				       （P = N）
end=>end: End
end1=>end: End
end2=>end: End

start->isNodeBlack
isNodeBlack(no)->end1
isNodeBlack(yes)->isChildLeaf
isChildLeaf(no)->handleCase1->end
isChildLeaf(yes)->isNodeNotRoot
isNodeNotRoot(yes)->isSiblingNodeBlack
isNodeNotRoot(no)->end2
isSiblingNodeBlack(no)->handleCase2->isNodeNotRoot
isSiblingNodeBlack(yes)->isSRNodeBlack
isSRNodeBlack(no)->handleCase3->end
isSRNodeBlack(yes)->isSLNodeBlack
isSLNodeBlack(no)->handleCase4->isNodeNotRoot
isSLNodeBlack(yes)->isParentNodeRed
isParentNodeRed(yes)->handleCase5->end
isParentNodeRed(no)->handleCase6->isNodeNotRoot

```

# 参考资料

[https://www.cnblogs.com/qingergege/p/7351659.html](https://www.cnblogs.com/qingergege/p/7351659.html)

[https://en.wikipedia.org/wiki/Red–black_tree](https://en.wikipedia.org/wiki/Red–black_tree)