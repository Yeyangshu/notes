# 享元模式

享元模式（Flyweight Pattern）是池技术的重要实现方式，可以降低大量重复的、细粒度的类在内存中的开销。

## 1 享元模式的定义

享元模式的英文原话是：

> Use sharing to support large numbers of fine-grained objects efficiently.

意思是：使用共享对象可有效地支持大量的细粒度的对象。享元模式是以共享的方式高效地支持大量的细粒度对象。享元对象能做到共享的关键是区分内部状态（Internal State）和外部状态（External State）。

- 内部状态是存储在享元对象内部的、可以共享的信息，并且不会随环境改变而改变。

- 外部状态是随环境改变而改变且不可以共享的状态。享元对象的外部状态必须由客户端保存，并在享元对象被创建之后，在需要使用的时候再传入到享元对象内部。



**重复利用对象**

**池的思想**

**Java string就是使用的享元模式**

**享元模式结合组合模式：画图软件的组合**



```java
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

/**
 * 子弹池，使用的时候从池子取出，改变living，不用新增
 */
public class BulletPool {
    List<Bullet> bullets = new ArrayList<>();

    {
        for (int i = 0; i < 5; i++) bullets.add(new Bullet());
    }

    /**
     * 从子弹池取出没有存活的对象（子弹池没有存活代表没有被使用）
     *
     * @return
     */
    public Bullet getBullet() {
        for (int i = 0; i < bullets.size(); i++) {
            Bullet b = bullets.get(i);
            if (!b.living) {
                return b;
            }
        }
        // 子弹池不够，新增子弹
        return new Bullet();
    }

    public static void main(String[] args) {
        BulletPool bp = new BulletPool();

        for (int i = 0; i < 10; i++) {
            Bullet b = bp.getBullet();
            System.out.println(b);
        }
    }

}

/**
 * 子弹
 */
class Bullet{

    /** 子弹唯一id */
    public UUID id = UUID.randomUUID();

    /** 子弹存活状态 */
    boolean living = true;

    @Override
    public String toString() {
        return "Bullet{" +
                "id=" + id +
                '}';
    }
}
```



# 