# LocaleResolver

![image-20210506204131618](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210506204131618.png)

默认四个实现类：

- AcceptHeaderLocaleResolver
- FixedLocaleResolver
- CookieLocaleResolver
- SessionLocaleResolver

AcceptHeaderLocaleResolver

```java
/**
 * LocaleResolver实现的抽象基类。 提供对默认语言环境的支持
 */
public abstract class AbstractLocaleResolver implements LocaleResolver {

	@Nullable
	private Locale defaultLocale;


	/**
	 * 设置默认的语言环境，如果未找到其他语言环境，此解析器将返回该语言环境。
	 */
	public void setDefaultLocale(@Nullable Locale defaultLocale) {
		this.defaultLocale = defaultLocale;
	}

	/**
	 * 返回此解析器应使用的默认语言环境（如果有）。
	 */
	@Nullable
	protected Locale getDefaultLocale() {
		return this.defaultLocale;
	}

}
```

