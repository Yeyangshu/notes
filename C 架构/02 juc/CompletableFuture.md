# CompletableFuture

## 实现异步 API

### 将同步方法转换为异步方法

Shop 类

```java
/**
 * 商店类
 *
 * @author yeyangshu
 * @version 1.0
 * @date 2021/1/2 23:07
 */
public class Shop {

    /**
     * 返回产品的价格，同步阻塞
     *
     * @param product 产品
     * @return 产品价格
     */
    public double getPrice(String product) {
        //
        return calculatePrice(product);
    }

    public Future<Double> getPriceAsync(String product) {
        CompletableFuture<Double> futurePrice = new CompletableFuture<>();
        new Thread(() -> {
            double price = calculatePrice(product);
            // 当请求的产品价格最终计算得出时，使用complete方法，结束completableFuture对象的运行，并设置变量的值。
            futurePrice.complete(price);
        }).start();
        return futurePrice;
    }

    private double calculatePrice(String product) {
        TimeUtils.delay();
        return new Random().nextDouble() * product.charAt(0) + product.charAt(1);
    }
    
}
```

客户端调用类

```java
public class Client {
    public static void main(String[] args) {
        Shop shop = new Shop();
        long start = System.nanoTime();
        Future<Double> futurePrice = shop.getPriceAsync("my favorite product");
        long invocationTime = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Invocation returned after " + invocationTime + " msecs");

        // 执行更多任务

        try {
            double price =  futurePrice.get();
            System.out.printf("Price is %.2f%n", price);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Price returned after " + retrievalTime + " msecs");
    }
}
```

`getPriceAsync` 返回时间

```
Invocation returned after 88 msecs
Price is 186.88
Price returned after 1109 msecs
```

`getPriceAsync` 方法的调用返回远远早于最终价格计算完成的时间。

### 错误处理

如果价格计算过程中产生了错误，用于提示错误的异常会被限制在试图计算商品价格的当前线程的范围内，最终会杀死该线程，而这会导致等待 get 方法返回结果的客户端永久地被阻塞。

一种值得推荐的做法：客户端可以使用重载版本的 get 方法，它使用一个超时参数来避免发生这样的情况。你应该尽量在你的代码中添加超时判断的逻辑，避免发生类似的问题。也因为如此，你没有机会发现计算商品价格的线程内到底发生了什么问题才引发了这样的失效。为了让客户端能了解商店无法提供请求商品价格的原因，你需要使用 CompletableFuture 的 completeExceptionally 方法将导致 CompletableFuture 内发生问题的异常抛出。

在代码中加入 1 / 0模拟异常

```java
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread(() -> {
        try {
            double price = calculatePrice(product);
            // 当请求的产品价格最终计算得出时，使用complete方法，结束completableFuture对象的运行，并设置变量的值。
            int i = 1 / 0;
            futurePrice.complete(price);
        } catch (Exception e) {
            // 如果价格计算正常，结束；否则就抛出失败的异常，完成这次Future的操作
            futurePrice.completeExceptionally(e);
        }
    }).start();
    return futurePrice;
}
```

测试结果

```
Invocation returned after 106 msecs
Exception in thread "main" java.lang.RuntimeException: java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
```

**使用工厂方法 supplyAsync 创建 CompletableFuture**

CompletableFuture 类自身提供了大量精巧的工厂方法，使用这些方法能更容易地完成整个流程，还不用担心实现的细节。比如，采用 supplyAsync 方法创建 CompletableFuture

```java
public Future<Double> getPriceAsync(String product) {
    return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

supplyAsync方法接受一个生产者（Supplier）作为参数，返回一个CompletableFuture对象，该对象完成异步执行后会读取调用生产者方法的返回值。生产者方法会交由ForkJoinPool池中的某个执行线程（Executor）运行，但是你也可以使用supplyAsync方法的重载版本，传递第二个参数指定不同的执行线程执行生产者方法。一般而言，向CompletableFuture的工厂方法传递可选参数，指定生产者方法的执行线程是可行的，

getPriceAsync 方法返回的 CompletableFuture 对象和上面手工创建和完成的 CompletableFuture 对象是完全等价的，这意味着它提供了同样的错误管理机制，而前者你花费了大量的精力才得以构建。

## 让你的代码免受阻塞之苦

采用顺序查询所有商店的方式实现的 findPrices 方法

```java
public class BestPriceFinder {
    static List<Shop> shops = Arrays.asList(new Shop("BestPrice"), new Shop("LetsSaveBig"),
            new Shop("MyFavoriteShop"), new Shop("BuyItAll"));

    public static List<String> findPrices(String product) {
        return shops.stream()
                .map(shop -> String.format("%s price is %.2f%n", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        long start = System.nanoTime();
        System.out.println(findPrices("myPhone27s"));
        long duration = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Done in " + duration + " msecs");

    }

```

结果：

```
[BestPrice price is 123.26
, LetsSaveBig price is 169.47
, MyFavoriteShop price is 214.13
, BuyItAll price is 184.74
]
Done in 4149 msecs
```

findPrices 方法的执行时间仅比4秒钟多了那么几毫秒，因为对这 4 个商店的查询是顺序进行的，并且一个查询操作会阻塞另一个，每一个操作都要花费大约 1 秒左右的时间计算请求商品的价格。

### 使用并行流对请求进行并行操作

```java
public class BestPriceFinder {
    static List<Shop> shops = Arrays.asList(new Shop("BestPrice"), new Shop("LetsSaveBig"),
            new Shop("MyFavoriteShop"), new Shop("BuyItAll"));

    public static List<String> findPrices(String product) {
        return shops.parallelStream()
                .map(shop -> String.format("%s price is %.2f%n", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        long start = System.nanoTime();
        System.out.println(findPrices("myPhone27s"));
        long duration = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Done in " + duration + " msecs");
    }
}
```

结果

```
[BestPrice price is 123.26
, LetsSaveBig price is 169.47
, MyFavoriteShop price is 214.13
, BuyItAll price is 184.74
]
Done in 1150 msecs
```

### 使用 CompletableFuture 发起异步请求

findPrices 方法中对不同商店的同步调用替换为异步调用

![image-20210103003718889](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210103003718889.png)

```java
public class BestPriceFinder {
    static List<Shop> shops = Arrays.asList(new Shop("BestPrice"), new Shop("LetsSaveBig"),
            new Shop("MyFavoriteShop"), new Shop("BuyItAll"));

    public static List<String> findPrices(String product) {
        List<CompletableFuture<String>> priceFuture = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(() -> String.format("%s price is %.2f%n", shop.getName(), shop.getPrice(product))))
                .collect(Collectors.toList());
        return priceFuture.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        long start = System.nanoTime();
        System.out.println(findPrices("myPhone27s"));
        long duration = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Done in " + duration + " msecs");
    }
}
```

结果

```
[BestPrice price is 123.26
, LetsSaveBig price is 169.47
, MyFavoriteShop price is 214.13
, BuyItAll price is 184.74
]
Done in 2137 msecs
```

### 使用定制的执行器

CompletableFuture 具有一定的优势，因为它允许你对执行器（Executor）进行配置，尤其是线程池的大小，让它以更适合应用需求的方式进行配置，满足程序的要求，而这是并行流API无法提供的。

```java
public class BestPriceFinder {
    private final List<Shop> shops = Arrays.asList(new Shop("BestPrice"), new Shop("LetsSaveBig"),
            new Shop("MyFavoriteShop"), new Shop("BuyItAll"), new Shop("ShopEasy"));

    private final Executor executor = Executors.newFixedThreadPool(Math.min(shops.size(), 100), new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setDaemon(true);
            return t;
        }
    });

    public List<String> findPrices(String product) {
        List<CompletableFuture<String>> priceFuture = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(() -> String.format("%s price is %.2f%n", shop.getName(), shop.getPrice(product)), executor))
                .collect(Collectors.toList());
        return priceFuture.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        BestPriceFinder bestPriceFinder = new BestPriceFinder();
        long start = System.nanoTime();
        System.out.println(bestPriceFinder.findPrices("myPhone27s"));
        long duration = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Done in " + duration + " msecs");
    }
}
```

结果

```java
[BestPrice price is 123.26
, LetsSaveBig price is 169.47
, MyFavoriteShop price is 214.13
, BuyItAll price is 184.74
, ShopEasy price is 176.08
]
Done in 1133 msecs
```

改进之后，使用 CompletableFuture 方案的程序处理 5 个商店仅耗时 1021 秒，处理 9 个商店时耗时 1022 秒。一般而言，这种状态会一直持续，直到商店的数目达到我们之前计算的阈值 400。这个例子证明了要创建更适合你的应用特性的执行器，利用 CompletableFutures 向其提交任务执行是个不错的主意。

目前为止，你已经知道对集合进行并行计算有两种方式：要么将其转化为并行流，利用 map 这样的操作开展工作，要么枚举出集合中的每一个元素，创建新的线程，在 CompletableFuture 内对其进行操作。后者提供了更多的灵活性，你可以调整线程池的大小，而这能帮助你确保整体的计算不会因为线程都在等待 I/O 而发生阻塞。我们对使用这些 API 的建议如下。

- 如果你进行的是计算密集型的操作，并且没有 I/O，那么推荐使用 Stream 接口，因为实现简单，同时效率也可能是最高的（如果所有的线程都是计算密集型的，那就没有必要创建比处理器核数更多的线程）。
- 反之，如果你并行的工作单元还涉及等待 I/O 的操作（包括网络连接等待），那么使用 CompletableFuture 灵活性更好，你可以像前文讨论的那样，依据等待/计算，或者 W/C 的比率设定需要使用的线程数。这种情况不使用并行流的另一个原因是，处理流的流水线中如果发生 I/O 等待，流的延迟特性会让我们很难判断到底什么时候触发了等待。

## 对多个异步任务进行流水线操作

```java
public List<String> findPricesSequential(String product) {
    return shops.stream()
        // 将每个shop对象转换成了一个字符串，该字符串包含了该shop中指定商品的价格和折扣代码。
        .map(shop -> shop.getPrice(product))
        // 对这些字符串进行解析，在Quote对象中对它们进行转换。
        .map(Quote::parse)
        // 操作联系远程的Discount服务，计算出最终的折扣价格，并返回该价格及提供该价格商品的shop。
        .map(Discount::applyDiscount)
        .collect(Collectors.toList());
}
```

结果

```
[BestPrice price is 110.93, LetsSaveBig price is 135.58, MyFavoriteShop price is 192.72, BuyItAll price is 184.74, ShopEasy price is 167.28]
sequential done in 10227 msecs
```

毫无意外，这次执行耗时 10 秒，因为顺序查询 5 个商店耗时大约 5 秒，现在又加上了 Discount 服务为 5 个商店返回的价格申请折扣所消耗的 5 秒钟。你已经知道，把流转换为并行流的方式，非常容易提升该程序的性能。不过，通过11.3节的介绍，你也知道这一方案在商店的数目增加时，扩展性不好，因为 Stream 底层依赖的是线程数量固定的通用线程池。相反，你也知道，如果自定义 CompletableFutures 调度任务执行的执行器能够更充分地利用 CPU 资源。

### 构造同步和异步操作