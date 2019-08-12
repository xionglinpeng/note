# `Spring知识点



## 方法注入

## 方法替换



## FactoryBean



## 通过编码的方式动态注册Bean

`DefaultListableBeanFactory`实现了`ConfigurableListableBeanFactory`接口，提供了可扩展配置、循环枚举的功能，可以通过此类实现Bean动态注入。为了实现在Spring容器启动阶段能动态注入自定义Bean，保证动态注入的Bean也能被AOP所增强，需要实现Bean工厂后置处理器接口`BeanFactoryPostProcessor`。

```java
@Component
public class UserServiceFactoryBean implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        //将ConfigurableListableBeanFactory强转为DefaultListableBeanFactory
        DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) configurableListableBeanFactory;
        //通过BeanDefinitionBuilder创建Bean定义
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(UserService1.class);
        //设置属性
        beanDefinitionBuilder.addPropertyValue("magic","黑暗穿刺");
        //注册bean定义
        beanFactory.registerBeanDefinition("userService1",beanDefinitionBuilder.getRawBeanDefinition());
        //直接注册一个bean实例
        beanFactory.registerSingleton("userService2",new UserService2());
    }
}
```



## BeanDefinition

## InstantiationStrategy

## BeanWrapper





## PropertyEditor





```java
package java.beans;
/**
 * @since 1.1
 */
public interface PropertyEditor {
    void setValue(Object value);
    Object getValue();
    //----------------------------------------------------------------------
    boolean isPaintable();
    void paintValue(java.awt.Graphics gfx, java.awt.Rectangle box);
    //----------------------------------------------------------------------
    String getJavaInitializationString();
    //----------------------------------------------------------------------
    String getAsText();
    void setAsText(String text) throws java.lang.IllegalArgumentException;
    //----------------------------------------------------------------------
    String[] getTags();
    //----------------------------------------------------------------------
    java.awt.Component getCustomEditor();
    boolean supportsCustomEditor();
    //----------------------------------------------------------------------
    void addPropertyChangeListener(PropertyChangeListener listener);
    void removePropertyChangeListener(PropertyChangeListener listener);
}
```

接口方法说明如下：

- `void setValue(Object value);`

  设置属性的值，基本类以封装类传入。

- `Object getValue();`

  返回当前属性的当前值，基本类型被封装成对应的封装类实例。

- `void setAsText(String text);`

  用一个字符串去更新属性内部值，这个字符串一般从外部属性编辑器传入。

- `String getAsText();`

  将属性对象用一个字符串表示，以便外部的属性编辑器能以可视化的方式先。默认返回null，表示该属性不能以字符串表示。

- `String[] getTags();`

  返回表示有效属性值的字符串数组（如boolean属性对应的有效Tag为true和false），以便属性编辑器能以下拉框的方式显示出来。默认放回null，表示属性没有匹配的字符串有限集合。

- `String getJavaInitializationString();`

  为属性提供一个表示初始值的字符串，属性编辑器以此值作为属性的默认值。

- `void addPropertyChangeListener(PropertyChangeListener listener);`

- `void removePropertyChangeListener(PropertyChangeListener listener);`

- `boolean supportsCustomEditor();`

- `java.awt.Component getCustomEditor();`

- `boolean isPaintable();`

- `void paintValue(java.awt.Graphics gfx, java.awt.Rectangle box);`





```java
package java.beans;

import java.beans.*;

/**
 * This is a support class to help build property editors.
 * <p>
 * It can be used either as a base class or as a delegate.
 *
 * @since 1.1
 */

public class PropertyEditorSupport implements PropertyEditor {

    /**
     * Constructs a {@code PropertyEditorSupport} object.
     *
     * @since 1.5
     */
    public PropertyEditorSupport() {
        setSource(this);
    }

    /**
     * @since 1.5
     */
    public PropertyEditorSupport(Object source) {
        if (source == null) {
           throw new NullPointerException();
        }
        setSource(source);
    }

    /**
     * @since 1.5
     */
    public Object getSource() {
        return source;
    }

    /**
     * @since 1.5
     */
    public void setSource(Object source) {
        this.source = source;
    }

    public void setValue(Object value) {
        this.value = value;
        firePropertyChange();
    }

    public Object getValue() {
        return value;
    }

    //----------------------------------------------------------------------

    public boolean isPaintable() {
        return false;
    }

    //----------------------------------------------------------------------

    public String getJavaInitializationString() {
        return "???";
    }

    //----------------------------------------------------------------------
    public String getAsText() {
        return (this.value != null)
                ? this.value.toString()
                : null;
    }
    public void setAsText(String text) throws java.lang.IllegalArgumentException {
        if (value instanceof String) {
            setValue(text);
            return;
        }
        throw new java.lang.IllegalArgumentException(text);
    }

    //----------------------------------------------------------------------
    public String[] getTags() {
        return null;
    }

    //----------------------------------------------------------------------
    public java.awt.Component getCustomEditor() {
        return null;
    }
    public boolean supportsCustomEditor() {
        return false;
    }

    //----------------------------------------------------------------------

    public synchronized void addPropertyChangeListener(
                                PropertyChangeListener listener) {
        if (listeners == null) {
            listeners = new java.util.Vector<>();
        }
        listeners.addElement(listener);
    }

    public synchronized void removePropertyChangeListener(
                                PropertyChangeListener listener) {
        if (listeners == null) {
            return;
        }
        listeners.removeElement(listener);
    }

    /**
     * Report that we have been modified to any interested listeners.
     */
    public void firePropertyChange() {
        java.util.Vector<PropertyChangeListener> targets;
        synchronized (this) {
            if (listeners == null) {
                return;
            }
            targets = unsafeClone(listeners);
        }
        // Tell our listeners that "everything" has changed.
        PropertyChangeEvent evt = new PropertyChangeEvent(source, null, null, null);

        for (int i = 0; i < targets.size(); i++) {
            PropertyChangeListener target = targets.elementAt(i);
            target.propertyChange(evt);
        }
    }

    @SuppressWarnings("unchecked")
    private <T> java.util.Vector<T> unsafeClone(java.util.Vector<T> v) {
        return (java.util.Vector<T>)v.clone();
    }

    //----------------------------------------------------------------------

    private Object value;
    private Object source;
    private java.util.Vector<PropertyChangeListener> listeners;
}

```











```java
package java.beans;
import java.awt.Image;

/**
 * @since 1.1
 */
public interface BeanInfo {
    BeanDescriptor getBeanDescriptor();
    EventSetDescriptor[] getEventSetDescriptors();
    int getDefaultEventIndex();
    PropertyDescriptor[] getPropertyDescriptors();
    int getDefaultPropertyIndex();
    MethodDescriptor[] getMethodDescriptors();
    BeanInfo[] getAdditionalBeanInfo();
    /**
     * @see #ICON_COLOR_16x16
     * @see #ICON_COLOR_32x32
     * @see #ICON_MONO_16x16
     * @see #ICON_MONO_32x32
     */
    Image getIcon(int iconKind);

    static final int ICON_COLOR_16x16 = 1;
    static final int ICON_COLOR_32x32 = 2;
    static final int ICON_MONO_16x16 = 3;

    /**
     * Constant to indicate a 32 x 32 monochrome icon.
     */
    static final int ICON_MONO_32x32 = 4;
}

```



### 注册PropertyEditor

```java
package org.springframework.beans.factory.config;
/**
 * {@link BeanFactoryPostProcessor} implementation that allows for convenient
 * registration of custom {@link PropertyEditor property editors}.
 *
 * <p>In case you want to register {@link PropertyEditor} instances,
 * the recommended usage as of Spring 2.0 is to use custom
 * {@link PropertyEditorRegistrar} implementations that in turn register any
 * desired editor instances on a given
 * {@link org.springframework.beans.PropertyEditorRegistry registry}. Each
 * PropertyEditorRegistrar can register any number of custom editors.
 *
 * <pre class="code">
 * &lt;bean id="customEditorConfigurer" class="org.springframework.beans.factory.config.CustomEditorConfigurer"&gt;
 *   &lt;property name="propertyEditorRegistrars"&gt;
 *     &lt;list&gt;
 *       &lt;bean class="mypackage.MyCustomDateEditorRegistrar"/&gt;
 *       &lt;bean class="mypackage.MyObjectEditorRegistrar"/&gt;
 *     &lt;/list&gt;
 *   &lt;/property&gt;
 * &lt;/bean&gt;
 * </pre>
 *
 * <p>
 * It's perfectly fine to register {@link PropertyEditor} <em>classes</em> via
 * the {@code customEditors} property. Spring will create fresh instances of
 * them for each editing attempt then:
 *
 * <pre class="code">
 * &lt;bean id="customEditorConfigurer" class="org.springframework.beans.factory.config.CustomEditorConfigurer"&gt;
 *   &lt;property name="customEditors"&gt;
 *     &lt;map&gt;
 *       &lt;entry key="java.util.Date" value="mypackage.MyCustomDateEditor"/&gt;
 *       &lt;entry key="mypackage.MyObject" value="mypackage.MyObjectEditor"/&gt;
 *     &lt;/map&gt;
 *   &lt;/property&gt;
 * &lt;/bean&gt;
 * </pre>
 *
 * <p>
 * Note, that you shouldn't register {@link PropertyEditor} bean instances via
 * the {@code customEditors} property as {@link PropertyEditor PropertyEditors} are stateful
 * and the instances will then have to be synchronized for every editing
 * attempt. In case you need control over the instantiation process of
 * {@link PropertyEditor PropertyEditors}, use a {@link PropertyEditorRegistrar} to register
 * them.
 *
 * <p>
 * Also supports "java.lang.String[]"-style array class names and primitive
 * class names (e.g. "boolean"). Delegates to {@link ClassUtils} for actual
 * class name resolution.
 *
 * <p><b>NOTE:</b> Custom property editors registered with this configurer do
 * <i>not</i> apply to data binding. Custom editors for data binding need to
 * be registered on the {@link org.springframework.validation.DataBinder}:
 * Use a common base class or delegate to common PropertyEditorRegistrar
 * implementations to reuse editor registration there.
 *
 * @author Juergen Hoeller
 * @since 27.02.2004
 * @see java.beans.PropertyEditor
 * @see org.springframework.beans.PropertyEditorRegistrar
 * @see ConfigurableBeanFactory#addPropertyEditorRegistrar
 * @see ConfigurableBeanFactory#registerCustomEditor
 * @see org.springframework.validation.DataBinder#registerCustomEditor
 */
public class CustomEditorConfigurer implements BeanFactoryPostProcessor, Ordered {

	protected final Log logger = LogFactory.getLog(getClass());

	private int order = Ordered.LOWEST_PRECEDENCE;  // default: same as non-Ordered

	@Nullable
	private PropertyEditorRegistrar[] propertyEditorRegistrars;

	@Nullable
	private Map<Class<?>, Class<? extends PropertyEditor>> customEditors;


	public void setOrder(int order) {
		this.order = order;
	}

	@Override
	public int getOrder() {
		return this.order;
	}

	public void setPropertyEditorRegistrars(PropertyEditorRegistrar[] propertyEditorRegistrars) {
		this.propertyEditorRegistrars = propertyEditorRegistrars;
	}

	public void setCustomEditors(Map<Class<?>, Class<? extends PropertyEditor>> customEditors) {
		this.customEditors = customEditors;
	}


	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		if (this.propertyEditorRegistrars != null) {
			for (PropertyEditorRegistrar propertyEditorRegistrar : this.propertyEditorRegistrars) {
				beanFactory.addPropertyEditorRegistrar(propertyEditorRegistrar);
			}
		}
		if (this.customEditors != null) {
			this.customEditors.forEach(beanFactory::registerCustomEditor);
		}
	}

}

```







Java原生的PropertyEditor存在以下不足：

- 只能用于字符串到Java对象的转换，不适用于任意两个Java类型之间的转换。
- 对源对象及目标对象所在的上下文信息（如注解、所在租住类的接口等）不敏感，在类型转换时不能利用这些上下文件信息实施高级的转换逻辑。

鉴于此，Spring在核心模块中添加了一个通用的类型转换模块，类型转换模块位于`org.springframework.core.convert`包中。Spring希望用这个类型转换体系替换Java标准的PropertyEditor。但由于历史原因，Spring将同时支持两者，在Bean配置、Spring MVC处理方法入参绑定中使用它们。











```java
package jdk.nashorn.internal.runtime;

public interface PropertyAccess {
    public int getInt(Object key, int programPoint);
    public int getInt(double key, int programPoint);
    public int getInt(int key, int programPoint);

    public double getDouble(Object key, int programPoint);
    public double getDouble(double key, int programPoint);
    public double getDouble(int key, int programPoint);

    public Object get(Object key);
    public Object get(double key);
    public Object get(int key);

    public void set(Object key, int value, int flags);
    public void set(Object key, double value, int flags);
    public void set(Object key, Object value, int flags);

    public void set(double key, int value, int flags);
    public void set(double key, double value, int flags);
    public void set(double key, Object value, int flags);
    
    public void set(int key, int value, int flags);
    public void set(int key, double value, int flags);
    public void set(int key, Object value, int flags);

    public boolean has(Object key);
    public boolean has(int key);
    public boolean has(double key);

    public boolean hasOwnProperty(Object key);
    public boolean hasOwnProperty(int key);
    public boolean hasOwnProperty(double key);

    public boolean delete(int key, boolean strict);
    public boolean delete(double key, boolean strict);
    public boolean delete(Object key, boolean strict);
}
```







## ConversionService

`ConversionService`是Spring 3.0版本提供，位于`org.springframework.core.convert`包。源码如下：

```java
public interface ConversionService {
	boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
	boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
	@Nullable
	<T> T convert(@Nullable Object source, Class<T> targetType);
	@Nullable
	Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
}

```

- `boolean canConvert(Class<?> sourceType, Class<?> targetType);`

  判断是否可以将一个Java类转换为另一个Java类。

- `boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);`

  需转换的类将以成员变量的方式出现在宿主类中。`TypeDescriptor `不但描述了需转换类的信息，还描述了从宿主类的上下文信息，如成员变量上的注解，成员变量是否以数组，集合或Map的方式呈现等。类型转换逻辑可以利用这些信息作出各种灵活的控制。

- `<T> T convert(Object source, Class<T> targetType);`

  将源类型对象转换为目标类型对象

- `Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);`

  将对象从源类型独享转换为目标类型对象，此时往往会用到所在宿主类的上下文信息。

### 自定义Converter

Sping提供了三种类型的转换器接口，它们都位于`org.springframework.core.convert.converter`包下：

- `org.springframework.core.convert.converter.Converter<S, T>`
- `org.springframework.core.convert.converter.ConverterFactory<S, R>`

- `org.springframework.core.convert.converter.GenericConverter`

#### `Converter<S, T>`

`Converter<S, T>`是Spring 3.0版本提供，位于`org.springframework.core.convert.converter`包。源码如下：

```java
@FunctionalInterface
public interface Converter<S, T> {
	@Nullable
	T convert(S source);
}
```

它是一个函数式接口。泛型S和T，即将S类型转换为T类型。

#### `ConverterFactory<S, R>`

`ConverterFactory<S, R>`接口的源码如下：

```java
package org.springframework.core.convert.converter;
public interface ConverterFactory<S, R> {
	<T extends R> Converter<S, T> getConverter(Class<T> targetType);
}
```

它是Spring 3.0版本提供的接口。具有泛型S、R和T，S为转换的源类型，R为目标类型的基类，而T为扩展于R基类的类型，即将S类型转换为T类型，而T类型必须为R类型或R类型的子类。

`ConverterFactory<S, R>`只有一个抽象方法`getConverter(Class<T> targetType);`，注意，它的返回值为`Converter<S, T>`类型，即它返回的不会转换后的类型数据，而是返回一个转换器进行实际的转换操作，从它的名字ConverterFactory就可以看出。

那么为什么要提供的这样的接口呢？

如果希望将一种类的对象转换为另一种类型及其子类的对象，举例来说，将String转换为Number及Number子类（Integer、Long、Double等）对象，就需要一系列的Converter，如StringToInteger、StringToLong及StringToDouble等。所以`ConverterFactory<S, R>`就是Spring提供的一个将相同系列多个“同质”Converter封装在一起的接口。

> ![](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/images/note.png)抽象方法`getConverter(Class<T> targetType);`的参数`targetType`就是要转换的目标类型。例如假设泛型`R`为`Number`类型，targetType就可以是Number以及Number的任何子类。具体实现可以参考Spring提供的`org.springframework.core.convert.support.StringToNumberConverterFactory`类。

#### `GenericConverter`
`GenericConverter`接口源码如下：

```java
package org.springframework.core.convert.converter;
import java.util.Set;
import org.springframework.core.convert.TypeDescriptor;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
public interface GenericConverter {
	@Nullable
	Set<ConvertiblePair> getConvertibleTypes();
	@Nullable
	Object convert(@Nullable Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
	final class ConvertiblePair {
		private final Class<?> sourceType;
		private final Class<?> targetType;
        //省略：构造、get方法、equals、hashCode、toString
		//......
	}
}

```

它是Spring 3.0版本提供的接口。



`ConvertiblePair`对象是`GenericConverter`的内部类，它封装了源类型和目标类型，组成一个“对子”。

`TypeDescriptor`包含了需要转换类型对象所在宿主类的信息。

##### `ConditionalGenericConverter`

`ConditionalGenericConverter`接口源码如下：

```java
package org.springframework.core.convert.converter;
public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {}
```

`ConditionalGenericConverter`继承了`GenericConverter`和`ConditionalConverter`接口。`ConditionalConverter`接口的源码如下：

```java
package org.springframework.core.convert.converter;
public interface ConditionalConverter {
	boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

该接口方法根据源类型及目标类型所在宿主类的上下文信息决定是否要进行类型转换，只有该接口方法返回true时，才调用`GenericConverter`的`convert()`方法完成类型转换。


### 注册Converter

Spring提供了一个FactoryBean用于注册`Converter`，这个FactoryBean是`org.springframework.context.support.ConversionServiceFactoryBean`，其源码如下：

```java
package org.springframework.context.support;
public class ConversionServiceFactoryBean implements FactoryBean<ConversionService>, InitializingBean {
   @Nullable
   private Set<?> converters;
    
   @Nullable
   private GenericConversionService conversionService;
    
   public void setConverters(Set<?> converters) {
      this.converters = converters;
   }
    
   @Override
   public void afterPropertiesSet() {
      this.conversionService = createConversionService();
      ConversionServiceFactory.registerConverters(this.converters, this.conversionService);
   }
    
   protected GenericConversionService createConversionService() {
      return new DefaultConversionService();
   }
    
   // implementing FactoryBean
   @Override
   @Nullable
   public ConversionService getObject() {
      return this.conversionService;
   }
    
   @Override
   public Class<? extends ConversionService> getObjectType() {
      return GenericConversionService.class;
   }
    
   @Override
   public boolean isSingleton() {
      return true;
   }
}
```

`ConversionServiceFactoryBean`有两个成员变量：

- `converter`：Converter转换器集合。
- `conversionService`：`ConversionService`Bean对象。

实现了`InitializingBean`，将调用`afterPropertiesSet()`方法完成`ConversionService`对象的创建以及`Converter`的注册，其注册是使用`org.springframework.core.convert.support.ConversionServiceFactory`注册的，它的实现非常简单，源码如下：

```java
package org.springframework.core.convert.support;
public final class ConversionServiceFactory {
	public static void registerConverters(@Nullable Set<?> converters, ConverterRegistry registry) {
		if (converters != null) {
			for (Object converter : converters) {
				if (converter instanceof GenericConverter) {
					registry.addConverter((GenericConverter) converter);
				}
				else if (converter instanceof Converter<?, ?>) {
					registry.addConverter((Converter<?, ?>) converter);
				}
				else if (converter instanceof ConverterFactory<?, ?>) {
					registry.addConverterFactory((ConverterFactory<?, ?>) converter);
				}
				else {
					throw new IllegalArgumentException("Each converter object must implement one of the " +
							"Converter, ConverterFactory, or GenericConverter interfaces");
				}
			}
		}
	}
}

```

`ConversionServiceFactoryBean`是FactoryBean，所以只需要如下配置即可完成自定义Converter的注册。

```xml
<bean class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.smart.domain.StringToUserConberter"/>
        </set>
    </property>
</bean>
```

在使用的时候只需要从Context容器中获取`ConversionService`Bean即可进行类型转换。



