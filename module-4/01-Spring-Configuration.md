## Application Context

Trong Spring, một đối tượng thực thi `ApplicationContext` interface là một container.

Application context trong một ứng dụng Spring là một đối tượng Java thực thi `ApplicationContext` interface và có trách nhiệm:
- Khởi tạo các bean trong application context.
- Cấu hình các bean trong application context.
- Kết hợp các bean trong application context.
- Quản lý vòng đời của các Spring bean.

Một số triển khai thường dùng của `ApplicationContext` interface bao gồm:

- **AnnotationConfigApplicationContext** - Application context dành cho ứng dụng độc lập sử dụng cấu hình dưới dạng các lớp có annotation.
    - Loại application context này được dùng cho các ứng dụng độc lập sử dụng cấu hình Java, nghĩa là các lớp được đánh dấu với `@Configuration`.
- **AnnotationConfigWebApplicationContext** - Tương tự như `AnnotationConfigApplicationContext`, nhưng dành cho các ứng dụng web.
- **ClassPathXmlApplicationContext** - Application context độc lập dùng cấu hình XML nằm trên `classpath` của ứng dụng.
- **FileSystemXmlApplicationContext** - Application context độc lập dùng cấu hình XML nằm dưới dạng một hoặc nhiều file trong `file system`.
- **XmlWebApplicationContext** - Application context cho ứng dụng web dùng cấu hình XML.

## Vòng đời của một Spring bean

Vòng đời của một Spring bean diễn ra như sau:
- Cấu hình Spring bean được đọc và metadata ở dạng đối tượng `BeanDefinition` được tạo cho mỗi bean.
    - Với cấu hình sử dụng annotation Java, `classpath` sẽ được quét bởi một bean loại `org.springframework.context.annotation.ClassPathBeanDefinitionScanner`, và các bean definition được đăng ký bởi một bean loại `org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader`.
- Tất cả các instance của `BeanFactoryPostProcessor` được gọi tuần tự và có cơ hội thay đổi metadata của bean.
- Đối với mỗi bean trong container:
    - Tạo một instance của bean sử dụng metadata của bean.
    - Cài đặt các thuộc tính và phụ thuộc của bean.
    - `BeanPostProcessor`'s `postProcessBeforeInitialization` có cơ hội xử lý instance mới của bean `trước khi khởi tạo`.
    - Bất kỳ phương thức nào trong lớp triển khai bean được đánh dấu với `@PostConstruct` sẽ được gọi.
        - `@PostConstruct` là một annotation vòng đời tiêu chuẩn của Java, như đã chỉ định trong JSR-250.
        - Phương thức được đánh dấu có thể có bất kỳ visibility nào, không nhận tham số và chỉ có kiểu trả về void.
    - Bất kỳ phương thức `afterPropertiesSet` nào trong lớp triển khai bean triển khai `InitializingBean` interface sẽ được gọi (Spring không khuyến nghị sử dụng phương thức này).
        - Mặc dù được sử dụng rộng rãi trong Spring framework, Spring Reference Documentation không khuyến nghị phương thức này do có thể gây phụ thuộc không cần thiết vào Spring framework.
    - Bất kỳ phương thức khởi tạo bean tùy chỉnh nào cũng sẽ được gọi. Phương thức khởi tạo bean có thể được chỉ định thông qua giá trị của thuộc tính `init-method` trong phần tử `<bean>` tương ứng trong cấu hình XML của Spring hoặc trong thuộc tính `initMethod` của annotation `@Bean`.
    - `BeanPostProcessor`'s `postProcessAfterInitialization` có cơ hội xử lý instance mới của bean `sau khi khởi tạo`.
    - Bean đã sẵn sàng để sử dụng.
- Khi application context của Spring tắt, các bean trong đó sẽ nhận các lệnh gọi hủy theo thứ tự này:
    - Bất kỳ phương thức nào trong lớp triển khai bean được đánh dấu với `@PreDestroy` sẽ được gọi.
        - `@PreDestroy` là một annotation vòng đời tiêu chuẩn của Java, như đã chỉ định trong JSR-250.
        - Phương thức được đánh dấu có thể có bất kỳ visibility nào, không nhận tham số và chỉ có kiểu trả về void.
    - Bất kỳ phương thức hủy nào trong lớp triển khai bean triển khai `DisposableBean` interface sẽ được gọi. Nếu cùng phương thức hủy đã được gọi, nó sẽ không được gọi lại nữa. (Spring không khuyến nghị sử dụng phương thức này)
        - Mặc dù được sử dụng rộng rãi trong Spring framework, Spring Reference Documentation không khuyến nghị phương thức này do có thể gây phụ thuộc không cần thiết vào Spring framework.
    - Bất kỳ phương thức hủy bean tùy chỉnh nào sẽ được gọi. Phương thức hủy bean có thể được chỉ định trong giá trị của thuộc tính `destroy-method` trong phần tử `<bean>` tương ứng trong cấu hình XML của Spring hoặc trong thuộc tính `destroyMethod` của annotation `@Bean`. Nếu cùng phương thức hủy đã được gọi, nó sẽ không được gọi lại nữa.

- **Phương thức khởi tạo** luôn được gọi khi một Spring bean được tạo, bất kể scope của bean là gì.
- Với các Spring bean có scope là prototype, nghĩa là một instance bean mới sẽ được tạo mỗi khi container của Spring nhận yêu cầu cho bean, **không có phương thức hủy** nào sẽ được gọi bởi Spring container.

### BeanFactoryPostProcessor

- Cấu hình bean definitions trước khi bean được tạo.
    - Có thể thay đổi `BeanDefinitions`
- `BeanFactoryPostProcessor` có thể tương tác và sửa đổi bean definitions, metadata của bean, nhưng không được tạo instance của bean.
- `BeanFactoryPostProcessor` là một interface định nghĩa thuộc tính (một phương thức duy nhất) của một loại điểm mở rộng container cho phép sửa đổi metadata của Spring bean trước khi khởi tạo các bean trong một container. Một bean factory post processor không thể tạo instance của bean, chỉ có thể sửa đổi metadata của bean. Một bean factory post processor chỉ được áp dụng cho metadata của các bean trong cùng một container mà nó được định nghĩa.
    - Ví dụ:
        - `PropertySourcesPlaceholderConfigurer` - cho phép chèn giá trị từ môi trường Spring hiện tại và các nguồn thuộc tính của môi trường này.
      ```java
        public class PropertySourcesPlaceholderConfigurer extends PlaceholderConfigurerSupport implements EnvironmentAware {
        
        }
      ```
        - `DeprecatedBeanWarner` - ghi lại cảnh báo về các bean có lớp triển khai được đánh dấu với annotation `@Deprecated`.
        - `PropertySourcesPlaceholderConfigurer` - là một `BeanFactoryPostProcessor` giải quyết các placeholder thuộc tính, ở dạng ${PROPERTY_NAME}, trong các thuộc tính của Spring bean và các thuộc tính của Spring bean được đánh dấu với annotation `@Value`.
        - Trong XML kích hoạt bằng `<context:property-placeholder location="classpath:db/db.properties" />`
    - Ví dụ về cách định nghĩa BPFP của riêng bạn

```java
public class DeprecationHandlerBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		final String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
		for (String name : beanDefinitionNames) {
			final BeanDefinition beanDefinition = beanFactory.getBeanDefinition(name);
			final String beanClassName = beanDefinition.getBeanClassName();
			try {
				final Class<?> beanClass = Class.forName(beanClassName);
				final DeprecatedClass annotation = beanClass.getAnnotation(DeprecatedClass.class);
				if (annotation != null) {
					beanDefinition.setBeanClassName(annotation.newImpl().getName());
				}
			}
			catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}
```

#### Static BeanFactoryPostProcessor

- `BeanFactoryPostProcessor` trả về bằng các phương thức `@Bean` là một vấn đề cần xem xét đặc biệt đối với các phương thức `@Bean` trả về loại `BeanFactoryPostProcessor` (BFPP) của Spring. Vì các đối tượng BFPP cần được khởi tạo rất sớm trong vòng đời của container, chúng có thể can thiệp vào xử lý các annotation như `@Autowired`, `@Value`, và `@PostConstruct` trong các lớp `@Configuration`.
  **Để tránh các vấn đề vòng đời này, đánh dấu các phương thức trả về BFPP `@Bean` là static.**
- Phương thức `@Bean` static có thể được định nghĩa để tạo, chẳng hạn như một `BeanFactoryPostProcessor` cần được khởi tạo trước khi khởi tạo bất kỳ bean nào mà `BeanFactoryPostProcessor` được dùng để sửa đổi trước khi các bean được sử dụng.
- Từ khóa 'static' cho phép phương thức được gọi mà không cần tạo lớp chứa cấu hình của nó như một instance.
- Điều này cần thiết khi các bean sẽ được khởi tạo sớm trong vòng đời của container.
- Nó giúp tránh kích hoạt các phần khác của cấu hình tại thời điểm định nghĩa.
- Chúng sẽ không bao giờ bị container chặn, ngay cả trong các lớp `@Configuration`.

```java
@Bean
public static PropertySourcesPlaceholderConfigurer pspc() {
  // khởi tạo, cấu hình và trả về pspc ...
}
```

### BeanPostProcessor

- `BeanPostProcessor` là một interface định nghĩa các phương thức callback cho phép sửa đổi instance của bean.
- `BeanPostProcessor` thông báo cho Spring rằng có một số xử lý mà Spring phải thực hiện sau khi khởi tạo bean. Spring thực thi các phương thức này cho mỗi bean.
- `BeanPostProcessor` được sử dụng để mở rộng chức năng của Spring.
- `BeanPostProcessor` có thể thay thế một instance của bean với, chẳng hạn, một AOP proxy.
    - Ví dụ
        - `AutowiredAnnotationBeanPostProcessor` - thực thi hỗ trợ cho dependency injection với annotation `@Autowired`.
        - `PersistenceExceptionTranslationPostProcessor` - áp dụng dịch lỗi cho các Spring bean được đánh dấu với annotation `@Repository`.
    - Ví dụ về cách định nghĩa BPP của riêng bạn

```java
public class DisplayNameBeanPostProcessor implements BeanPostProcessor {

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("Called postProcessBeforeInitialization " + beanName);
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("Called postProcessAfterInitialization " + beanName);
		return bean;
	}
}
```

- Khi định nghĩa một `BeanPostProcessor` bằng một phương thức `@Bean` có annotation,
  khuyến nghị rằng phương thức nên là static để post-processor được khởi tạo sớm trong quá trình tạo Spring context.
    - Từ khóa `static` cho phép phương thức được gọi mà không cần tạo lớp chứa cấu hình của nó như một instance.
    - Điều này cần thiết khi các bean sẽ được khởi tạo sớm trong vòng đời của container.
    - Nó giúp tránh kích hoạt các phần khác của cấu hình tại thời điểm định nghĩa

## Initializing beans priority

```java
public class TriangleLifecycle implements InitializingBean {

	//  Calls #1
	@PostConstruct
	public void postConstruct(){
		System.out.println("TriangleLifecycle postConstruct : " + toString());
	}

	// Calls #2
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("TriangleLifecycle afterPropertiesSet : " + toString());
	}

	// Calls #3
	public void initMethod(){
		System.out.println("TriangleLifecycle initMethod : " + toString());
	}

}
```

## @javax.annotation.PostConstruct

- Annotation `@PostConstruct` là một phần của JSR-250 và được dùng trên một phương thức cần thực thi **sau khi hoàn tất việc tiêm dependency** để thực hiện khởi tạo.
- Bean đăng ký `@PostConstruct` là `org.springframework.context.annotation.CommonAnnotationBeanPostProcessor`.
    - Khi tạo một Spring application context sử dụng cấu hình dựa trên annotation, chẳng hạn `AnnotationConfigApplicationContext`, một `CommonAnnotationBeanPostProcessor` mặc định sẽ tự động được đăng ký trong application context và không cần thêm cấu hình nào để kích hoạt `@PostConstruct` và `@PreDestroy`.
- Các phương thức được đánh dấu bằng `@PostConstruct` phải tuân thủ các quy tắc sau:
    - Không có tham số
    - Phải trả về kiểu `void`
    - Có thể có bất kỳ mức độ truy cập nào

- **Giao dịch** (*Transaction*) chưa được cấu hình hoặc chưa tồn tại khi `@PostConstruct` được xử lý, do đó không nên gọi nguồn dữ liệu (*data source*) trong phương thức `@PostConstruct`.
    - Vì `@PostConstruct` được thực hiện trước khi mọi proxy được cấu hình.
        - Các proxy sẽ được cấu hình trong giai đoạn `postProcessAfterInitialization` trong `BeanPostProcessor` (BPP), nhưng `@PostConstruct` được xử lý trước giai đoạn `postProcessAfterInitialization` trong BPP.

Điều này đảm bảo rằng `@PostConstruct` có thể thực hiện các tác vụ khởi tạo cần thiết nhưng không nên xử lý các thành phần phụ thuộc vào giao dịch hoặc các proxy khác do thời điểm khởi tạo sớm của nó trong vòng đời bean.

