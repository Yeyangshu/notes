# AssertUtil

## 1 自定义异常

### 1.1 AbstractCommonException：抽象父类

```java
/**
 * 异常抽象父类，其他异常继承此类
 */
public abstract class AbstractCommonException extends RuntimeException {

    /**
     * 构造函数
     */
    public AbstractCommonException() {
    }

    /**
     * 构造函数
     *
     * @param cause 异常原因
     */
    public AbstractCommonException(Throwable cause) {
        super(cause);
    }

    /**
     * 构造函数
     *
     * @param msg 异常描述
     */
    public AbstractCommonException(String msg) {
        super(msg);
    }

    /**
     * 构造函数
     *
     * @param msg   异常描述
     * @param cause 异常原因
     */
    public AbstractCommonException(String msg, Throwable cause) {
        super(msg, cause);
    }

}
```

### 1.2 各系统实现此接口自定义的异常类

```java
public class OrderException extends AbstractCommonException {

    /** 错误码 */
    private CommonResultCode code;

    /**
     * 构造函数
     *
     * @param message 输出信息
     */
    public OrderException(String message) {
        super(message);
        this.code = OrderResultEnum.SYSTEM_EXCEPTION;
    }

    /**
     * 构造函数
     *
     * @param message 输出信息
     * @param cause   异常原因
     */
    public OrderException(String message, Throwable cause) {
        super(message, cause);
        this.code = OrderResultEnum.SYSTEM_EXCEPTION;
    }

    /**
     * 构造函数
     *
     * @param code    返回码
     * @param message 输出信息
     */
    public OrderException(OrderResultEnum code, String message) {
        super(message);
        this.code = code;
    }

    /**
     * 构造函数
     *
     * @param code    返回码
     * @param message 输出信息
     * @param cause   异常原因
     */
    public OrderException(OrderResultEnum code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    /**
     * 设置返回码
     *
     * @param resultCode 返回码
     */
    public void setResultCode(OrderResultEnum resultCode) {
        this.code = resultCode;
    }

    /**
     * 获取订单系统结果码
     *
     * @return 订单系统结果吗
     */
    public OrderResultEnum getCode() {
        if (code instanceof OrderResultEnum) {
            return (OrderResultEnum) code;
        }
        return null;
    }

}
```

## 2 自定义返回码

### 2.1 CommonResultCode：通用返回码接口

```java
/**
 * 通用返回码接口
 */
public interface CommonResultCode {

    /**
     * 获取返回码的code。
     *
     * @return 结果码
     */
    public String getResultCode();

    /**
     * 获取返回码的描述信息。
     *
     * @return 描述信息
     */
    public String getResultMsg();

}
```

### 2.2 自定义返回码

```java
/**
 * 订单系统错误码
 */
public enum OrderResultEnum implements CommonResultCode {

    /** 订单成功 */
    ORDER_SUCCESS("ORDER_SUCCESS", "订单成功"),

    /** 订单失败 */
    ORDER_FAIL("ORDER_FAIL", "订单失败"),

    /** 系统异常 */
    SYSTEM_EXCEPTION("SYSTEM_EXCEPTION", "系统异常");

    /** 枚举编号 */
    private final String code;

    /** 枚举详情 */
    private final String description;

    /** 系统名  */
    private static final String SYSTEM_NAME = "order";

    /**
     * 构造方法
     *
     * @param code 枚举编号
     * @param description 枚举详情
     */
    private OrderResultEnum(String code, String description) {
        this.code = code;
        this.description = description;
    }

    /** 错误码列表 */
    private static final List<OrderResultEnum> orderResultEnumList = new ArrayList<>();

    static {
        // 加载所有枚举至内存
        for (OrderResultEnum orderResultEnum : values()) {
            orderResultEnumList.add(orderResultEnum);
        }
    }

    /**
     * 获取错误码列表
     *
     * @return 错误码列表
     */
    public static List<OrderResultEnum> getOrderResultEnumList() {
        return orderResultEnumList;
    }

    /**
     * 根据错误码返回枚举
     * @param resultCode 错误码
     * @return OrderResultEnum
     */
    public static OrderResultEnum getByResultCode(String resultCode) {
        for (OrderResultEnum rc : values()) {
            if (StringUtils.equals(rc.getResultCode(), resultCode)) {
                return rc;
            }
        }
        return null;
    }

    @Override
    public String getResultCode() {
        return getCode();
    }

    @Override
    public String getResultMsg() {
        return this.description;
    }

    public String getCode() {
        return code;
    }
	
    // 自己重写
    @Override
    public String toString() {
        return "IfinfluxResultCode{" +
                "code='" + code + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}
```

## 3 自定义AssertUtil

### 3.1 AssertUtil

```java
/**
 * 自定义断言工具类
 */
public class AssertUtil {

    /** 自定义异常类名称 */
    private static String exceptionClassName;

    /** 异常对象构造方法 */
    private static Constructor constructor;

    public static void isTrue(final boolean expValue, final OrderResultEnum resultCode, final Object... objects) {
        if (!expValue) {
            OrderException exception = null;
            String logString = getLogString(objects);
            String resultMsg = StringUtils.isBlank(logString) ? resultCode.getResultMsg() : logString;
            try {
                exception = (OrderException) constructor.newInstance(resultMsg);
            } catch (Throwable e) {
                throw new IllegalArgumentException("AssertUtil has not been initiallized correctly![exceptionClassName="
                                + exceptionClassName + ",resultCode=" + resultCode + ",resultMsg="
                                + resultMsg + "]", e);
            }
            exception.setResultCode(resultCode);
            throw exception;
        }
    }

    public static void notNull(final Object object, final OrderResultEnum resultCode, final Object... objs) {
        isTrue(object != null, resultCode, objs);
    }

    /**
     * 动态输入参数转换为字符串
     *
     * @param objects 动态参数
     * @return 字符串
     */
    public static String getLogString(Object... objects) {
        StringBuilder outLog = new StringBuilder();
        for (Object inputLog : objects) {
            outLog.append(inputLog);
        }
        return outLog.toString();
    }

    /**
     *  setter方法
     *
     * @param exceptionClassName 异常类名称
     */
    public void setExceptionClassName(String exceptionClassName) {
        AssertUtil.exceptionClassName = exceptionClassName;
        initConfig();
    }

    /**
     * 初始化AssertUtil配置。
     */
    @SuppressWarnings("unchecked")
    private static void initConfig() {

        Class expClassTemp = null;

        // 1 加载异常类
        if (StringUtils.isBlank(exceptionClassName)) {
            throw new IllegalArgumentException("exceptionClassName has not set!");
        }

        try {
            expClassTemp = Class.forName(exceptionClassName);
        } catch (Throwable e) {
            throw new IllegalArgumentException("loading exceptionClass failed![exceptionClassName=" + exceptionClassName + "]", e);
        }

        // 必须是AssertException的子类
        if (!AbstractCommonException.class.isAssignableFrom(expClassTemp)) {
            throw new IllegalArgumentException("illegal exceptionClass type, must be the subclass of CommonException![exceptionClassName="
                    + exceptionClassName + "]");
        }

        Constructor constructorTemp = null;

        // 2 获取构造方法
        try {
            constructorTemp = expClassTemp.getConstructor(String.class);
        } catch (Throwable e) {
            throw new IllegalArgumentException("constructor method not found![exceptionClassName=" + exceptionClassName + "]", e);
        }

        // 3 缓存反射结果
        constructor = constructorTemp;
    }
}
```

### 3.2 AssertUtil配置

#### 3.2.1 xml文件

```xml
<bean class="com.yeyangshu.study.assertutil.AssertUtil">
    <property name="exceptionClassName" value="com.yeyangshu.assertutil.OrderException"/>
</bean>
```

#### 3.2.2 Configuration

```java
@Configuration
public class AssertUtilConfiguration {

    @Bean
    public AssertUtil assertUtil() {
        AssertUtil assertUtil = new AssertUtil();
        assertUtil.setExceptionClassName("com.yeyangshu.assertutil.OrderException");
        return assertUtil;
    }
}
```

## 4 测试

`SpringBoot + Junit5` 测试

代码

```java
@SpringBootTest
class StudyApplicationTests {

	@Test
	public void assertTest() {
		
		System.out.println("--------order is success--------");
		
		boolean orderSuccess = true;
		AssertUtil.isTrue(orderSuccess, OrderResultEnum.ORDER_FAIL, "order is fail");

		System.out.println("--------order is fail--------");

		boolean orderFail = false;
//		AssertUtil.isTrue(orderFail, OrderResultEnum.ORDER_FAIL, "order is fail");

		System.out.println("--------order is fail, use order exception --------");
		AssertUtil.isTrue(orderFail, OrderResultEnum.ORDER_FAIL);

	}
}
```

结果

```java
/**
 * --------order is success--------
 * --------order is fail--------
 * --------order is fail, use order exception --------
 * com.yeyangshu.assertutil.OrderException: 订单失败
 */
```