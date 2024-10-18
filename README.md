
还记得上一篇中我们遗留的问题吗？我们再简要回顾一下，现在有一颗空的二叉查找树，我们分别插入1，2，3，4，5，五个节点，那么得到的树是什么样子呢？这个不难想象，二叉树如下：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144747210-1971647322.png)


树的高度是4，并且数据结构上和链表没有区别，查找性能也和链表一致。如果我们将树的结构改变一下呢？比如改成下面的树结构，


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144754570-1482648967.png)


那么树的高度变成了3，并且它也是一棵二叉查找树，树的高度越低，查找性能就越高，这是我们理想中的数据结构。如果想要树的高度尽可能的低，那么左右子树的高度差就不能相差太多。这就引出了我们今天的主题**AVL平衡二叉树**，AVL平衡二叉树的定义为任意节点的左右子树的高度差不能超过1。这样就可以保证我们的这棵树的高度保持在一个最低的状态，这样我们的查找性能也是最优的。那么我们如何在树的变化时（也就是增加节点或删除节点时），保证AVL平衡二叉树的性质呢？下面我们就针对每一种情况进行分析。


#### 左左单旋转


我们先看看下面的例子，以下每一个例子都是最复杂的情况，完全覆盖简单的情况，所以我们把最复杂情况用代码实现了，那么简单的情况也会涵盖在内。看下图


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144802739-984782326.png)


上图中，原本以k1为根节点的树是一个AVL平衡二叉树，这时，我们向树中插入节点2，根据二叉查找树的性质，最后节点2插入的位置如上图。插入节点后，我们每个节点分析一下，看看节点是否还符合AVL平衡二叉树的性质。我们先看看节点3，插入节点2后，节点3的左子树的高度是0，因为只有一个节点2。再看节点3的右子树，右子树为空，那么高度为\-1，**这里我们统一规定，如果节点为空，那么高度为\-1**。节点3的左右子树高度为1，符合AVL平衡二叉树的性质，同理我们再看节点k2，左子树高度为1，右子树高度为0，高度差为1，也符合AVL平衡二叉树。再看节点k1，左子树k2的高度为2，右子树的高度为0，相差为2，所以在节点k1处不满足AVL平衡二叉树的性质，我们要进行调整，使得以k1为根节点的树变为一个AVL平衡二叉树，我们要怎么做呢？


由于左子树的高度比较高，所以我们要将树旋转一下，用k2作根节点，k1作为k2的右子节点，旋转后如图所示：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144812899-604264359.png)


旋转后，以k2为根节点的新树，是一棵AVL平衡二叉树。这里我们要**特别注意一下节点5的位置**，它的原始位置是k2的右子树，而k2又是k1的左子树，根据二叉查找树的性质，k2的右子树中的值是大于k2，小于k1的。旋转后，k2变成了根节点，k1变成k2的右子树，那么原k2的右子树（节点5），变为k1的左子树。那么这棵树根据二叉查找树的性质，还是大于k2，小于k1的，没有变动，这是符合我们的预期的。通过上述的旋转，我们得到的新树是一棵AVL平衡二叉树。


我们总结一下重要的点，为编码做准备：


1. 发现k1的左子树比右子树高度大于1；
2. 发现k1的左子树k2的左子树高度大于k2的右子树高度，这种称作**左\-左情形**。要做左侧单旋转。
3. 将k2作为新树的节点，k2的右子树改为k1，k1的左子树改为k2的右子树。
4. 更新k1和k2的高度。


完成上面的操作，我们得到一个新的AVL平衡二叉树。下面我们进入具体编码。



```
/**
 * 二叉树节点
 * @param 
 */
public class BinaryNodeextends Comparable> {

    //节点数据
    @Setter@Getter
    private T element;
    //左子节点
    @Setter@Getter
    private BinaryNode left;
    //右子节点
    @Setter@Getter
    private BinaryNode right;
    //节点高度
    @Setter@Getter
    private Integer height;

    //构造函数
    public BinaryNode(T element) {
        if (element == null) {
            throw new RuntimeException("二叉树节点元素不能为空");
        }
        this.element = element;
        this.height = 0;
    }
}

```

我们现在改造`BinaryNode`类，并在类中增加高度属性，高度默认为0。



```
/**
 * 二叉查找树
 */
public class BinarySearchTreeextends Comparable> {
    ……
    /**
     * 插入元素
     *
     * @param element
     */
    public void insert(T element) {
        root = insert(root, element);
    }

    private BinaryNode insert(BinaryNode tree, T element) {
        if (tree == null) {
            tree = new BinaryNode<>(element);
        } else {
            int compareResult = element.compareTo(tree.getElement());
            if (compareResult > 0) {
                tree.setRight(insert(tree.getRight(), element));
            }

            if (compareResult < 0) {
                tree.setLeft(insert(tree.getLeft(), element));
            }
        }

        return balance(tree);
    }
    
    /**
     * 平衡节点
     * @param tree
     */
    private BinaryNode balance(BinaryNode tree) {
        if (tree == null) {
            return null;
        }
        Integer leftHeight = height(tree.getLeft());
        Integer rightHeight = height(tree.getRight());
        if (leftHeight - rightHeight > 1) {
            //左-左情形，单旋转
            if (height(tree.getLeft().getLeft()) >= height(tree.getLeft().getRight())) {
                tree = rotateWithLeftChild(tree);
            }
        } 

        //当前节点的高度 = 最高的子节点 + 1
        tree.setHeight(Math.max(leftHeight,rightHeight) + 1);
        return tree;
    }
        
	/**
     * 节点的高度
     * @param node
     * @return
     */
    public Integer height(BinaryNode node) {
        return node == null?-1:node.getHeight();
    }   
    
    /**
     * 左侧单旋转
     * @param k1
     */
    private BinaryNode rotateWithLeftChild(BinaryNode k1) {
        BinaryNode k2 = k1.getLeft();
        k1.setLeft(k2.getRight());
        k2.setRight(k1);
        k1.setHeight(Math.max(height(k1.getLeft()),height(k1.getRight()))+1);
        k2.setHeight(Math.max(height(k2.getLeft()),height(k2.getRight()))+1);
        return k2;
    }
    ……
}

```

我们再在`BinarySearchTree`类中增加height方法，获取节点的高度，如果节点为空，返回\-1。由于`insert`后，树可能会发生旋转，节点会发生变化，所以这里，`insert`方法改造为会有返回值。在第一个`insert`方法中，调用第二个`insert`方法，并用root去接第二个insert方法的返回值，说明整棵树的根节点可能会发生旋转变化。同样在第二个insert方法中，递归调用时，根据不同的条件，将返回值给到当前节点的左或右子节点。节点插入完成后，我们统一调用`balance`方法，如果节点不满足平衡条件，我们要进行相应的旋转，最后把相关的节点的高度进行更新，这个`balance`方法是我们今天重点的方法。


进入balance方法后，我们分别获取左右子树的高度，如果左子树的高度比右子树高度大于1，说明不满足平衡条件，需要进行旋转。然后再判断左子树的左子树与左子树的右子树的高度，如果大于，说明是`左-左情形`，需要左侧单旋转。这里比较绕，大家多看几篇，加深理解。我们把以当前节点为根节点的子树传入`rotateWithLeftChild`方法中，为了和上面的图对应起来，变量的名称叫做k1。那么对应的k2就是k1的左子树，然后进行旋转，k1的左子树设置为k2的右子树，k2的右子树设置为k1，然后再重新计算k1和k2的高度，最后将k2作为新子树的根节点返回。这样`左-左情形`的单旋转就实现了。我们可以多看几遍代码加深一下理解。


#### 右右单旋转


与左左相对称的是`右-右情形`，我们看下图：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144828250-232060636.png)


我们插入节点6后，导致以k1为根节点的子树不平衡，需要进行旋转，旋转的动作与左左情形完全对称，总结操作如下：


1. 发现k1的右子树比左子树的高度大于1；
2. 发现k1的右子树k2的右子树高度大于k2的左子树高度，这种称作**右\-右情形**。要做右侧单旋转。
3. 将k2作为新树的节点，k2的左子树改为k1，k1的右子树改为k2的左子树。
4. 更新k1和k2的高度。


旋转后，如下图：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144840165-695723739.png)


我们按照上面的操作进行编码，



```
/**
 * 平衡节点
 * @param tree
 */
private BinaryNode balance(BinaryNode tree) {
    if (tree == null) {
        return null;
    }
    Integer leftHeight = height(tree.getLeft());
    Integer rightHeight = height(tree.getRight());
    if (leftHeight - rightHeight > 1) {
        //左-左情形，单旋转
        if (height(tree.getLeft().getLeft()) >= height(tree.getLeft().getRight())) {
            tree = rotateWithLeftChild(tree);
        }
    } else if (rightHeight - leftHeight > 1){
        //右-右情形，单旋转
        if (height(tree.getRight().getRight()) >= height(tree.getRight().getLeft())) {
            tree = rotateWithRightChild(tree);
        }
    }

    //当前节点的高度 = 最高的子节点 + 1
    tree.setHeight(Math.max(leftHeight,rightHeight) + 1);
    return tree;
}

/**
 * 右侧单旋转
 * @param k1
 * @return
 */
private BinaryNode rotateWithRightChild(BinaryNode k1) {
    BinaryNode k2 = k1.getRight();
    k1.setRight(k2.getLeft());
    k2.setLeft(k1);
    k1.setHeight(Math.max(height(k1.getLeft()),height(k1.getRight()))+1);
    k2.setHeight(Math.max(height(k2.getLeft()),height(k2.getRight()))+1);

    return k2;
}

```

在balance方法中，我们增加了`右-右情形`的判断，然后调用`rotateWithRightChild`方法，在这个方法中，为了和上图对应，变量的名字我们依然叫做k1和k2。k1的右节点设置为k2的左节点，k2的左节点设置为k1，然后更新高度，最后把新的根节点k2返回。


#### 左右双旋转


下面我们再看双旋转的情形，如下图所示：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144852886-457797919.png)


我们新插入节点3后，导致以k1为根节点的子树不满足平衡条件，我们先用之前的左侧单旋转，看看能不能满足，如下图所示：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144901121-1777445695.png)


旋转后，以k2为根节点的新树，右子树比左子树的高度大于1，**也不满足平衡条件，所以这种方案是不行的**。那我们要怎么做呢？我们只有将k3作为新的根节点才能满足平衡条件，将k3移动到根节点我们需要旋转两次，第一次先在k2节点进行右旋转，将k3旋转到k1的左子节点的位置，如图：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144912867-1581707725.png)


然后再在k1位置进行左旋转，将k3移动到根节点，如图：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144921462-744097296.png)


这样就满足了平衡条件，细心的小伙伴可能注意到了，原k3的做节点挂到了k2的右节点上，原k3的右节点刮到了k1的左节点上。这些细节并不需要我们特殊的处理，因为在左旋转右旋转的方法中已经处理过了，我们再总结一下具体的细节：


1. 插入节点后，发现k1的左子树比右子树高度大于1；
2. 发现k1的左子树k2，k2的右子树比k2的左子树高，这是`左-右情形`，需要双旋转。
3. 将k1的左子树k2进行右旋转；
4. 将k1进行左旋转；


我们编码实现



```
/**
 * 平衡节点
 * @param tree
 */
private BinaryNode balance(BinaryNode tree) {
    if (tree == null) {
        return null;
    }
    Integer leftHeight = height(tree.getLeft());
    Integer rightHeight = height(tree.getRight());
    if (leftHeight - rightHeight > 1) {
        //左-左情形，单旋转
        if (height(tree.getLeft().getLeft()) >= height(tree.getLeft().getRight())) {
            tree = rotateWithLeftChild(tree);
        } else {// 左-右情形，双旋转
            tree = doubleWithLeftChild(tree);
        }
    } else if (rightHeight - leftHeight > 1){
        //右-右情形，单旋转
        if (height(tree.getRight().getRight()) >= height(tree.getRight().getLeft())) {
            tree = rotateWithRightChild(tree);
        }
    }

    //当前节点的高度 = 最高的子节点 + 1
    tree.setHeight(Math.max(leftHeight,rightHeight) + 1);
    return tree;
}

/**
 * 左侧双旋转
 * @param k1
 * @return
 */
private BinaryNode doubleWithLeftChild(BinaryNode k1) {
    k1.setLeft(rotateWithRightChild(k1.getLeft()));
    return rotateWithLeftChild(k1);
}

```

我们在balance方法中，增加`左-右情形`的判断，然后调用`doubleWithLeftChild`方法，在这个方法中，我们按照之前总结的步骤，先将k1的左节点进行一次右旋转，然后再将k1进行左旋转，最后将新的根节点返回，旋转后达到了平衡的条件。


#### 右左双旋转


最后我们再来看与左右情形对称的`右-左情形`，树的初始结构如下图：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144932749-398796196.png)


插入节点8后，导致k1节点的右子树高度比左子树高度大于1，同时k2的左子树比右子树高，这就是`右-左情形`。这时，我们需要先在k2节点做一次左旋转，旋转后如图：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144939745-373106718.png)


然后再在k1节点做一次右旋转，旋转后如图：


![](https://img2024.cnblogs.com/blog/1191201/202410/1191201-20241017144945561-104861962.png)


我们参照上面的左右情形，总结一下右左情形的操作：


1. 插入节点后，发现k1的右子树比左子树高度大于1；
2. 发现k1的右子树k2，k2的左子树比k2的右子树高，这是`右-左情形`，需要双旋转。
3. 将k1的右子树k2进行左旋转；
4. 将k1进行右旋转；


然后我们编码实现：



```
/**
 * 平衡节点
 * @param tree
 */
private BinaryNode balance(BinaryNode tree) {
    if (tree == null) {
        return null;
    }
    Integer leftHeight = height(tree.getLeft());
    Integer rightHeight = height(tree.getRight());
    if (leftHeight - rightHeight > 1) {
        //左-左情形，单旋转
        if (height(tree.getLeft().getLeft()) >= height(tree.getLeft().getRight())) {
            tree = rotateWithLeftChild(tree);
        } else {// 左-右情形，双旋转
            tree = doubleWithLeftChild(tree);
        }
    } else if (rightHeight - leftHeight > 1){
        //右-右情形，单旋转
        if (height(tree.getRight().getRight()) >= height(tree.getRight().getLeft())) {
            tree = rotateWithRightChild(tree);
        } else {//右-左情形，双旋转
            tree = doubleWithRightChild(tree);
        }
    }

    //当前节点的高度 = 最高的子节点 + 1
    tree.setHeight(Math.max(leftHeight,rightHeight) + 1);
    return tree;
}

/**
 * 右侧双旋转
 * @param k1
 * @return
 */
private BinaryNode doubleWithRightChild(BinaryNode k1) {
    k1.setRight(rotateWithLeftChild(k1.getRight()));
    return rotateWithLeftChild(k1);
}

```

由于左右单旋转的方法在之前已经实现过了，所以双旋转的实现，我们直接调用就可以了，先将k1的右节点进行一次左旋转，再将k1进行右旋转，最后返回新的根节点。因为节点的高度正在左右单旋转的方法里已经处理了，所以这里不需要特殊的处理。


#### 删除节点


与插入节点一样，删除节点也会引起树的不平衡，同样，在删除节点后，我们调用balance方法使树再平衡。remove改造方法如下：



```
/**
 * 删除元素
 * @param element
 */
public void remove(T element) {
    root = remove(root, element);
}

private BinaryNode remove(BinaryNode tree, T element) {
    if (tree == null) {
        return null;
    }
    int compareResult = element.compareTo(tree.getElement());
    if (compareResult > 0) {
        tree.setRight(remove(tree.getRight(), element));
    } else if (compareResult < 0) {
        tree.setLeft(remove(tree.getLeft(), element));
    }
    if (tree.getLeft() != null && tree.getRight() != null) {
        tree.setElement(findMin(tree.getRight()));
        tree.setRight(remove(tree.getRight(), tree.getElement()));
    } else {
        tree = tree.getLeft() != null ? tree.getLeft() : tree.getRight();
    }
    return balance(tree);
}

```

同样，remove方法会引起子树根节点的变化，所以，第二个remove方法要增加返回值，在调用第二个remove方法时，要用返回值覆盖当前的节点。


#### 总结


好了，AVL平衡二叉树的操作就完全实现了，它解决了树的不平衡问题，使得查询效率大幅提升。小伙伴们有问题，欢迎评论区留言\~\~


 本博客参考[MeoMiao 萌喵加速](https://biqumo.org)。转载请注明出处！
