# 组合模式

**树状结构专用模式**

组合模式（Composite Pattern）也叫合成模式，用来描述部分与整体的关系。

## 1 组合模式的定义

组合模式的英文原话是：

> Compose objects into tree structures to represent part-whole hierarchies.Composite lets clients treat individual objects and compositions of objectsuniformly.

意思是：将对象组合成**树形结构**以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性。

组合模式提供以下3个角色：

- 抽象构件（Component）角色：该角色定义参加组合对象的共有方法和属性，规范一些默认的行为接口。
- 叶子构件（Leaf）角色：该角色是叶子对象，其下没有其他的分支，定义出参加组合的原始对象的行为。

- 树枝构件（Composite）角色：该角色代表参加组合的、其下有分支的树枝对象，它的作用是将树枝和叶子组合成一个树形结构，并定义出管理子对象的方法，如add()、remove()等。



测试

```java
import java.util.ArrayList;
import java.util.List;

public class Client {
    public static void main(String[] args) {
        BranchNode root = new BranchNode("root");
        BranchNode chapter1 = new BranchNode("chapter1");
        BranchNode chapter2 = new BranchNode("chapter2");
        Node r1 = new LeafNode("r1");
        Node c11 = new LeafNode("c11");
        Node c12 = new LeafNode("c12");
        BranchNode b21 = new BranchNode("section21");
        Node c211 = new LeafNode("c21");
        Node c212 = new LeafNode("c22");

        root.add(chapter1);
        root.add(chapter2);
        root.add(r1);
        chapter1.add(c11);
        chapter1.add(c12);
        chapter2.add(b21);
        b21.add(c211);
        b21.add(c212);

        tree(root, 0);

    }

    /**
     * root
     * --chapter1
     * ----c11
     * ----c12
     * --chapter2
     * ----section21
     * ------c21
     * ------c22
     * --r1
     */

    /**
     * 递归函数
     *
     * @param b 节点
     * @param depth 节点深度
     */
    static void tree(Node b, int depth) {
        for (int i = 0; i < depth; i++) {
            System.out.print("--");
        }
        b.print();

        if (b instanceof BranchNode) {
            for (Node n : ((BranchNode) b).nodes) {
                tree(n, depth + 1);
            }
        }
    }
    
}

/**
 * 抽象节点
 */
abstract class Node {
    public abstract void print();
}

/**
 * 叶子节点
 */
class LeafNode extends Node {
    String context;
    @Override
    public void print() {
        System.out.println(context);
    }

    /**
     * 构造函数
     *
     * @param context
     */
    public LeafNode(String context) {
        this.context = context;
    }
}

/**
 * 分支节点
 */
class BranchNode extends Node {

    /** 节点列表 */
    List<Node> nodes = new ArrayList<>();

    /** 节点名称 */
    String name;

    @Override
    public void print() {
        System.out.println(name);
    }

    /**
     * 添加节点
     *
     * @param node
     */
    public void add(Node node) {
        nodes.add(node);
    }

    /**
     * 构造函数
     *
     * @param name
     */
    public BranchNode(String name) {
        this.name = name;
    }

}
```


