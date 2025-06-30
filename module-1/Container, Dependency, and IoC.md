## Tổng Hợp Các Câu Hỏi Thường Gặp Về Spring Framework

Dưới đây là danh sách các câu hỏi và câu trả lời chi tiết về các khái niệm cốt lõi trong Spring Framework, giúp bạn hiểu rõ hơn về cách Spring hoạt động.

### Câu hỏi 01: Dependency Injection là gì và ưu điểm của nó là gì?

**Dependency Injection (DI)** là một kỹ thuật lập trình phần mềm trong đó các đối tượng không tự tạo ra các dependency (sự phụ thuộc) của chúng. Thay vào đó, các đối tượng khai báo các dependency mà chúng cần, và một đối tượng bên ngoài hoặc một framework sẽ có nhiệm vụ cung cấp các dependency cụ thể cho các đối tượng đó.

**Các loại Dependency Injection:**

* Constructor injection
* Setter injection
* Interface injection

**Ưu điểm của việc sử dụng Dependency Injection:**

* Tăng khả năng tái sử dụng mã nguồn
* Tăng khả năng đọc hiểu mã nguồn
* Tăng khả năng bảo trì mã nguồn
* Tăng khả năng kiểm thử (testability) của mã nguồn
* Giảm sự kết dính (coupling)
* Tăng sự gắn kết (cohesion)

#### Ví dụ Code

Hãy xem xét một ví dụ đơn giản về một lớp `OrderService` phụ thuộc vào một `NotificationService`.

**Cách làm truyền thống (Không dùng DI):**
Lớp `OrderService` tự tạo ra đối tượng `EmailService`. Điều này làm cho chúng bị kết dính chặt chẽ.

```java
// Lớp phụ thuộc
public class EmailService {
    public void sendNotification(String message) {
        System.out.println("Sending Email: " + message);
    }
}

// Lớp OrderService tự tạo dependency
public class OrderService {
    private final EmailService notificationService;

    public OrderService() {
        // Tự tạo dependency, gây ra kết dính chặt
        this.notificationService = new EmailService();
    }

    public void placeOrder() {
        // ... logic đặt hàng
        notificationService.sendNotification("Your order has been placed!");
    }
}
```

**Sử dụng Dependency Injection:**
Framework (ví dụ: Spring) sẽ "tiêm" một implementation của `NotificationService` vào `OrderService` thông qua hàm khởi tạo.

```java
// Định nghĩa một interface chung
public interface NotificationService {
    void sendNotification(String message);
}

// Một implementation cụ thể
@Component("emailService")
public class EmailService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        System.out.println("Sending Email: " + message);
    }
}

// Một implementation khác
@Component("smsService")
public class SmsService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

// OrderService nhận dependency từ bên ngoài
@Component
public class OrderService {
    private final NotificationService notificationService;

    // Dependency được "tiêm" vào qua constructor
    @Autowired
    public OrderService(@Qualifier("emailService") NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void placeOrder() {
        // ... logic đặt hàng
        notificationService.sendNotification("Your order has been placed!");
    }
}
```

### Câu hỏi 02: Pattern là gì? Anti-pattern là gì? Dependency Injection có phải là một pattern không?

* **Software Design Pattern (Mẫu thiết kế phần mềm)** là một giải pháp có thể tái sử dụng cho một vấn đề thường xuyên xảy ra trong thiết kế phần mềm. Nó là một mô tả cấp cao về cách giải quyết vấn đề và có thể được sử dụng trong nhiều tình huống khác nhau. Các mẫu thiết kế thường đại diện cho các phương pháp hay nhất (best practices) mà các nhà phát triển có thể sử dụng.
* **Anti-pattern** là một giải pháp không hiệu quả và phản tác dụng cho một vấn đề thường xảy ra.
* **Dependency Injection là một pattern.** Nó giải quyết vấn đề về việc tạo ra các dependency một cách linh hoạt.

#### Ví dụ Code: Anti-pattern "God Object"

**God Object** là một anti-pattern trong đó một đối tượng biết quá nhiều hoặc làm quá nhiều việc. Dưới đây là ví dụ.

**Anti-pattern:**

```java
// Một lớp làm tất cả mọi thứ: quản lý người dùng, sản phẩm, đơn hàng
public class GodObject {
    public void createUser() { /* ... */ }
    public void deleteUser() { /* ... */ }
    public void addProduct() { /* ... */ }
    public void removeProduct() { /* ... */ }
    public void createOrder() { /* ... */ }
    public void shipOrder() { /* ... */ }
    // ... và nhiều phương thức khác
}
```

**Cách làm tốt hơn (sử dụng Pattern):** Chia nhỏ trách nhiệm ra các lớp khác nhau.

```java
// Chỉ quản lý người dùng
public class UserService {
    public void createUser() { /* ... */ }
    public void deleteUser() { /* ... */ }
}

// Chỉ quản lý sản phẩm
public class ProductService {
    public void addProduct() { /* ... */ }
    public void removeProduct() { /* ... */ }
}

// Chỉ quản lý đơn hàng
public class OrderService {
    public void createOrder() { /* ... */ }
    public void shipOrder() { /* ... */ }
}
```

### Câu hỏi 03: Interface là gì và ưu điểm của việc sử dụng chúng trong Java là gì? Tại sao chúng được khuyên dùng cho Spring bean?

Trong Java, **interface** là một kiểu tham chiếu, chứa một tập hợp các phương thức trừu tượng. Một lớp khi `implement` một interface phải triển khai tất cả các phương thức của interface đó, hoặc phải tự khai báo mình là một lớp trừu tượng.

**Ưu điểm của việc sử dụng interface trong Java:**

* Cho phép tách rời (decoupling) giữa "hợp đồng" (contract) và "cách triển khai" (implementation).
* Cho phép khai báo hợp đồng giữa bên gọi (callee) và bên được gọi (caller).
* Tăng khả năng hoán đổi (interchangeability).
* Tăng khả năng kiểm thử (testability).

**Tại sao interface được khuyên dùng cho Spring bean:**

* **Cho phép sử dụng JDK Dynamic Proxy**.
* **Cho phép ẩn giấu implementation**.
* **Cho phép dễ dàng chuyển đổi bean**.

#### Ví dụ Code

Ví dụ đã được trình bày trong Câu hỏi 01, nơi `OrderService` phụ thuộc vào `NotificationService` (một interface), và chúng ta có thể dễ dàng thay đổi giữa `EmailService` và `SmsService` mà không cần sửa đổi `OrderService`.

### Câu hỏi 04: Khái niệm "application-context" có nghĩa là gì?

**ApplicationContext** là một phần trung tâm của ứng dụng Spring. Nó chứa các định nghĩa về bean và là nơi đăng ký các thành phần của ứng dụng. Nó cho phép bạn truy xuất các bean đã được cấu hình và lắp ráp.

**ApplicationContext thực hiện các nhiệm vụ:**

* Khởi tạo Beans
* Cấu hình Beans
* Lắp ráp (Assemblies) Beans
* Quản lý vòng đời của Beans
* Là một `BeanFactory`
* Là một `ResourceLoader`
* Có khả năng đẩy các sự kiện đến các listener đã đăng ký
* Expose `Environment` cho phép giải quyết các thuộc tính (properties)

#### Ví dụ Code

Cách tạo và sử dụng một `ApplicationContext` để lấy một bean.

```java
// Lớp cấu hình Spring
@Configuration
@ComponentScan("com.example.myapp")
public class AppConfig {
    // Các định nghĩa @Bean có thể ở đây
}

// Bean sẽ được quản lý bởi Spring
@Component
public class MyService {
    public void doSomething() {
        System.out.println("Service is doing something.");
    }
}

// Lớp chính để chạy ứng dụng
public class MainApp {
    public static void main(String[] args) {
        // 1. Tạo một ApplicationContext từ lớp cấu hình
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        // 2. Lấy một bean từ context
        MyService myService = context.getBean(MyService.class);
        
        // 3. Sử dụng bean
        myService.doSomething();
    }
}
```

### Câu hỏi 05: Khái niệm "container" là gì và vòng đời của nó như thế nào?

**Container** là một môi trường thực thi cung cấp các dịch vụ kỹ thuật bổ sung cho mã nguồn của bạn sử dụng. Thông thường, các container sử dụng kỹ thuật IoC (Inversion of Control), cho phép bạn tập trung vào việc tạo ra khía cạnh nghiệp vụ của mã nguồn, trong khi các khía cạnh kỹ thuật như chi tiết giao tiếp được cung cấp bởi môi trường thực thi. Spring cung cấp một container cho các bean, quản lý vòng đời của chúng và cung cấp các dịch vụ bổ sung thông qua `ApplicationContext`.

**Vòng đời của Spring Container:**

1.  Ứng dụng được khởi động.
2.  Spring container được tạo ra.
3.  Container đọc cấu hình.
4.  Các định nghĩa bean (bean definitions) được tạo ra từ cấu hình.
5.  Các `BeanFactoryPostProcessor` xử lý các định nghĩa bean.
6.  Các instance của Spring bean được tạo ra.
7.  Các Spring bean được cấu hình và lắp ráp - giải quyết các giá trị thuộc tính và tiêm các dependency.
8.  Các `BeanPostProcessor` được gọi.
9.  Ứng dụng chạy.
10. Ứng dụng bị tắt.
11. Spring Context được đóng lại.
12. Các callback hủy (destruction callbacks) được gọi.

### Câu hỏi 06: Bạn sẽ tạo một instance mới của ApplicationContext như thế nào?

Cách tạo một `ApplicationContext` phụ thuộc vào loại ứng dụng của bạn.

* **Đối với ứng dụng không phải web (Non-Web Applications):**

    * `AnnotationConfigApplicationContext`
    * `ClassPathXmlApplicationContext`
    * `FileSystemXmlApplicationContext`

* **Đối với ứng dụng web (Web Applications):**

    * `XmlWebApplicationContext`
    * `AnnotationConfigWebApplicationContext`

* **Đối với Spring Boot:**

    * Spring Boot tự động quản lý việc tạo `ApplicationContext` cho bạn.

#### Ví dụ Code

**Sử dụng `ClassPathXmlApplicationContext`:**

`applicationContext.xml` (đặt trong `src/main/resources`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myService" class="com.example.MyService"/>

</beans>
```

`Main.java`:

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        MyService service = context.getBean("myService", MyService.class);
        service.doWork();
    }
}
```

**Sử dụng `AnnotationConfigApplicationContext`:**

`AppConfig.java`:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

`Main.java`:

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MyService service = context.getBean(MyService.class);
        service.doWork();
    }
}
```

### Câu hỏi 07: Bạn có thể mô tả vòng đời của một Spring Bean trong ApplicationContext không?

Vòng đời của một bean mô tả các giai đoạn mà bean trải qua từ khi được khởi tạo cho đến khi bị hủy.

1.  **Context được tạo ra**:

    1.  Các định nghĩa bean được tạo dựa trên cấu hình Spring.
    2.  Các `BeanFactoryPostProcessor` được gọi.

2.  **Bean được tạo ra**:

    1.  Instance của bean được tạo.
    2.  Các thuộc tính và dependency được thiết lập (tiêm vào).
    3.  Phương thức `postProcessBeforeInitialization` của `BeanPostProcessor` được gọi.
    4.  Phương thức được chú thích `@PostConstruct` được gọi.
    5.  Phương thức `afterPropertiesSet` của interface `InitializingBean` được gọi.
    6.  Phương thức `initMethod` tùy chỉnh được gọi.
    7.  Phương thức `postProcessAfterInitialization` của `BeanPostProcessor` được gọi.

3.  **Bean sẵn sàng để sử dụng**.

4.  **Bean bị hủy**:

    1.  Phương thức được chú thích `@PreDestroy` được gọi.
    2.  Phương thức `destroy` của interface `DisposableBean` được gọi.
    3.  Phương thức `destroyMethod` tùy chỉnh được gọi.

#### Ví dụ Code

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class LifecycleBean implements InitializingBean, DisposableBean {

    public LifecycleBean() {
        System.out.println("1. Constructor called");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("2. @PostConstruct method called");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("3. afterPropertiesSet (InitializingBean) called");
    }

    public void customInit() {
        System.out.println("4. Custom init-method called");
    }

    // Bean is now ready and in use...

    @PreDestroy
    public void preDestroy() {
        System.out.println("5. @PreDestroy method called");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("6. destroy (DisposableBean) called");
    }

    public void customDestroy() {
        System.out.println("7. Custom destroy-method called");
    }
}

// Cấu hình trong lớp @Configuration:
@Configuration
public class AppConfig {
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public LifecycleBean lifecycleBean() {
        return new LifecycleBean();
    }
}
```

### Câu hỏi 08: Bạn sẽ tạo một ApplicationContext trong một integration test như thế nào?

Để tạo một `ApplicationContext` cho các bài kiểm thử tích hợp (integration tests), bạn nên sử dụng các tính năng hỗ trợ kiểm thử của Spring.

1.  **Thêm dependency `spring-test`:**
    Đảm bảo rằng bạn có dependency `spring-test` trong file `pom.xml` hoặc `build.gradle` với scope là `test`.

2.  **Sử dụng các annotation của Spring Test:**

    * Sử dụng `@RunWith(SpringRunner.class)` (cho JUnit 4) hoặc `@ExtendWith(SpringExtension.class)` (cho JUnit 5).
    * Sử dụng `@ContextConfiguration` để chỉ định cách tải `ApplicationContext`.

#### Ví dụ Code (sử dụng JUnit 5)

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import static org.junit.jupiter.api.Assertions.assertNotNull;

// Lớp cấu hình cho bài test
@Configuration
class TestConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}

// Lớp test
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = TestConfig.class)
public class MyServiceIntegrationTest {

    @Autowired
    private MyService myService;

    @Test
    public void testServiceIsNotNull() {
        assertNotNull(myService);
        myService.doSomething();
    }
}
```

### Câu hỏi 09: Cách được ưu tiên để đóng một application context là gì? Spring Boot có tự làm điều này cho bạn không?

**Đối với ứng dụng độc lập không phải web (Standalone Non-Web Applications):**

* **Cách được khuyên dùng:** Đăng ký một `shutdown hook` bằng cách gọi `ConfigurableApplicationContext#registerShutdownHook()`.
* **Cách khác:** Gọi trực tiếp `ConfigurableApplicationContext#close()`.

**Đối với ứng dụng web (Web Application):**

* `ContextLoaderListener` sẽ tự động đóng context.

**Đối với Spring Boot:**

* **Có**, Spring Boot sẽ tự động đóng `ApplicationContext` cho bạn. Một `shutdown hook` sẽ được tự động đăng ký.

#### Ví dụ Code (Ứng dụng Standalone)

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MainApp {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        // Đăng ký một shutdown hook để đảm bảo context được đóng khi ứng dụng kết thúc
        context.registerShutdownHook();

        MyService service = context.getBean(MyService.class);
        service.doWork();
    }
}
```

### Câu hỏi 10: Mô tả các khái niệm sau đây

#### 1\. Dependency injection sử dụng Java configuration

Khi sử dụng `@Configuration`, bạn định nghĩa các bean bằng các phương thức `@Bean`. Để tiêm dependency, bạn thêm các tham số vào phương thức `@Bean`, và Spring sẽ tự động tìm và tiêm các bean tương ứng.

**Ví dụ Code:**

```java
@Configuration
public class ApplicationConfiguration {

    @Bean
    public SpringBean1 springBean1(SpringBean2 springBean2, SpringBean3 springBean3) {
        return new SpringBean1(springBean2, springBean3);
    }

    @Bean
    public SpringBean2 springBean2() {
        return new SpringBean2();
    }

    @Bean
    public SpringBean3 springBean3() {
        return new SpringBean3();
    }
}
```

#### 2\. Dependency injection sử dụng annotations (@Component, @Autowired)

* Đánh dấu các lớp bằng `@Component`, `@Service`, v.v.
* Kích hoạt component scanning bằng `@ComponentScan`.
* Sử dụng `@Autowired` trên các trường, constructor, hoặc phương thức setter để tiêm dependency.

**Ví dụ Code:**

```java
@Component
public class SpringBean2 { /* ... */ }

@Component
public class SpringBean1 {
    @Autowired
    private SpringBean2 springBean2;
}

@Configuration
@ComponentScan(basePackages = "com.example.beans")
public class ApplicationConfiguration {
}
```

#### 3\. Component scanning, Stereotypes và Meta-Annotations

* **Component Scanning**: Là quá trình Spring quét classpath để tìm các lớp được đánh dấu bằng stereotype và tạo bean từ chúng.
* **Stereotypes**: Là các annotation mô tả vai trò của lớp (`@Component`, `@Service`, `@Repository`, `@Controller`).
* **Meta-Annotations**: Là các annotation có thể được dùng để tạo ra các annotation khác. Ví dụ, `@RestController` là sự kết hợp của `@Controller` và `@ResponseBody`.

#### 4\. Scopes cho Spring beans? Scope mặc định là gì?

**Scope** của bean định nghĩa vòng đời và khả năng hiển thị của instance. Các scope phổ biến:

* **singleton**: Chỉ một instance duy nhất cho mỗi container. **Đây là scope mặc định.**
* **prototype**: Một instance mới mỗi khi được yêu cầu.
* **request**: Một instance mới cho mỗi HTTP request.
* **session**: Một instance mới cho mỗi HTTP session.
* **application**: Một instance cho mỗi `ServletContext`.
* **websocket**: Một instance cho mỗi WebSocket session.

**Ví dụ Code:**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

@Configuration
public class AppConfig {

    @Bean
    @Scope("singleton")
    public MySingletonBean mySingletonBean() {
        return new MySingletonBean();
    }

    @Bean
    @Scope("prototype")
    public MyPrototypeBean myPrototypeBean() {
        return new MyPrototypeBean();
    }
}
```

### Câu hỏi 11: Mặc định, các bean được khởi tạo một cách lười biếng (lazily) hay hăng hái (eagerly)? Làm thế nào để thay đổi hành vi này?

Theo mặc định, các bean `Singleton` được khởi tạo một cách hăng hái (eagerly). Ngược lại, các bean `Prototype` được khởi tạo một cách lười biếng (lazily), nghĩa là instance chỉ được tạo khi có yêu cầu bean đó. Tuy nhiên, nếu một bean `Singleton` có phụ thuộc vào một bean `Prototype`, thì instance của bean `Prototype` đó sẽ được tạo ra một cách hăng hái để đáp ứng các phụ thuộc cho bean `Singleton`.

**Cách thay đổi hành vi:**

Bạn có thể thay đổi hành vi mặc định cho tất cả các bean bằng cách sử dụng annotation `@ComponentScan`.

* `@ComponentScan(lazyInit = true)`: Sẽ làm cho tất cả các bean được khởi tạo lười biếng, kể cả các bean `Singleton`.
* `@ComponentScan(lazyInit = false)`: Đây là giá trị mặc định, sẽ tạo các bean `Singleton` một cách hăng hái và các bean `Prototype` một cách lười biếng.

Bạn cũng có thể thay đổi hành vi cho từng bean cụ thể bằng cách sử dụng annotation `@Lazy`.

* Annotation `@Lazy` nhận một tham số để quyết định việc khởi tạo lười biếng có nên xảy ra hay không.
* Mặc định, `@Lazy` được sử dụng để đánh dấu một bean sẽ được khởi tạo lười biếng.
* Bạn có thể sử dụng `@Lazy(false)` để buộc một bean phải được khởi tạo hăng hái, ngay cả khi đã thiết lập `@ComponentScan(lazyInit = true)` trên toàn cục.
* `@Lazy` có thể được áp dụng trên:
    * Các lớp được chú thích bằng `@Component`.
    * Các lớp được chú thích bằng `@Configuration`, làm cho tất cả các bean do lớp cấu hình này cung cấp trở nên lười biếng.
    * Các phương thức được chú thích bằng `@Bean`, làm cho bean được tạo bởi phương thức đó trở nên lười biếng.

#### Ví dụ Code

```java
// EagerBean sẽ được khởi tạo ngay khi ứng dụng khởi động
@Component
public class EagerBean {
    public EagerBean() {
        System.out.println("EagerBean has been created!");
    }
}

// LazyBean chỉ được khởi tạo khi nó được yêu cầu lần đầu tiên
@Component
@Lazy
public class LazyBean {
    public LazyBean() {
        System.out.println("LazyBean has been created!");
    }
}

@Configuration
@ComponentScan
public class AppConfig {
}

// Lớp chính để chạy ứng dụng
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        System.out.println("Spring context started.");
        
        // Tại thời điểm này, EagerBean đã được tạo, nhưng LazyBean thì chưa.
        // Chỉ khi dòng lệnh dưới đây được thực thi, LazyBean mới được tạo.
        LazyBean lazyBean = context.getBean(LazyBean.class);
        System.out.println("LazyBean requested from context.");
    }
}
```

### Câu hỏi 12: Property source là gì? Bạn sẽ sử dụng @PropertySource như thế nào?

`PropertySource` là một lớp trừu tượng của Spring về các cặp khóa-giá trị trong môi trường (Environment). Các nguồn này có thể đến từ:

* Thuộc tính JVM
* Biến môi trường hệ thống
* Thuộc tính JNDI
* Tham số Servlet
* Tệp properties nằm trên hệ thống tệp
* Tệp properties nằm trên classpath

Bạn đọc các thuộc tính bằng cách sử dụng annotation `@PropertySource` hoặc `@PropertySources`. Để truy cập các thuộc tính, bạn sử dụng annotation `@Value`.

#### Ví dụ Code

Giả sử bạn có một tệp `app.properties` trong `src/main/resources`:

```properties
app.name=My Awesome Application
app.version=1.0.0
```

Bạn có thể tải và sử dụng các thuộc tính này như sau:

```java
@Configuration
@PropertySource("classpath:app.properties") // Tải tệp properties từ classpath
public class AppConfig {
}

@Component
public class AppInfo {
    private final String appName;
    private final String appVersion;

    // Sử dụng @Value để tiêm các giá trị thuộc tính
    public AppInfo(@Value("${app.name}") String appName, @Value("${app.version}") String appVersion) {
        this.appName = appName;
        this.appVersion = appVersion;
    }

    public void printInfo() {
        System.out.println("Application Name: " + appName);
        System.out.println("Application Version: " + appVersion);
    }
}
```

### Câu hỏi 13: BeanFactoryPostProcessor là gì và nó được dùng để làm gì? Khi nào nó được gọi?

`BeanFactoryPostProcessor` là một interface cho phép bạn tạo logic để sửa đổi Siêu dữ liệu (Metadata) của Spring Bean trước khi bất kỳ Bean nào được tạo ra. Nó không tạo ra bất kỳ bean nào, nhưng nó có thể truy cập và thay đổi Metadata được sử dụng sau này để tạo ra các Bean.

`BeanFactoryPostProcessor` được gọi sau khi Spring đã đọc hoặc phát hiện các Định nghĩa Bean (Bean Definitions), nhưng trước khi bất kỳ Spring Bean nào được tạo ra.

Vì `BeanFactoryPostProcessor` là một loại bean đặc biệt cần được gọi trước các loại bean khác, Spring cần có khả năng tạo nó trước bất kỳ bean nào khác. Đây là lý do tại sao `BeanFactoryPostProcessor` cần được đăng ký từ một phương thức `static`.

`PropertySourcesPlaceholderConfigurer` là một `BeanFactoryPostProcessor` được sử dụng để giải quyết các placeholder thuộc tính (ví dụ: `${property_name}`) trong các Spring Bean trên các trường được chú thích bằng `@Value`.

#### Ví dụ Code

Ví dụ về một `BeanFactoryPostProcessor` tùy chỉnh để thay đổi một thuộc tính của bean definition.

```java
public class CustomBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("CustomBeanFactoryPostProcessor is invoked.");
        
        // Lấy định nghĩa của một bean cụ thể
        BeanDefinition beanDefinition = beanFactory.getBeanDefinition("myService");
        
        // Sửa đổi metadata, ví dụ: thay đổi scope của bean thành prototype
        System.out.println("Changing 'myService' bean scope to prototype.");
        beanDefinition.setScope("prototype");
    }
}

@Configuration
public class AppConfig {
    @Bean
    public static CustomBeanFactoryPostProcessor customBeanFactoryPostProcessor() {
        return new CustomBeanFactoryPostProcessor();
    }
    
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

### Câu hỏi 14: BeanPostProcessor là gì và nó khác với BeanFactoryPostProcessor như thế nào?

`BeanPostProcessor` là một interface cho phép bạn tạo các phần mở rộng cho Spring Framework để sửa đổi các đối tượng Spring Bean trong quá trình khởi tạo. Interface này chứa hai phương thức: `postProcessBeforeInitialization` và `postProcessAfterInitialization`.

Sự khác biệt chính so với `BeanFactoryPostProcessor` là `BeanFactoryPostProcessor` làm việc với **Định nghĩa Bean (Bean Definitions)**, trong khi `BeanPostProcessor` làm việc với **Đối tượng Bean (Bean Objects)**.

**Phương thức khởi tạo (initialization method)** là một phương thức bạn có thể viết cho một Spring Bean nếu bạn cần thực hiện một số mã khởi tạo phụ thuộc vào các thuộc tính và/hoặc các dependency đã được tiêm vào Spring Bean. Bạn có thể khai báo phương thức khởi tạo theo ba cách:

* Tạo một phương thức trong Spring Bean được chú thích bằng `@PostConstruct`.
* Implement phương thức `afterPropertiesSet` của `InitializingBean`.
* Sử dụng `@Bean(initMethod = "...")` trong lớp Configuration.

**Phương thức hủy (destroy method)** là một phương thức trong Spring Bean mà bạn có thể sử dụng để triển khai bất kỳ logic dọn dẹp nào cho các tài nguyên mà Bean sử dụng. Phương thức này sẽ được gọi khi Spring Bean không còn được sử dụng, thường là khi Spring Context bị đóng. Bạn có thể khai báo phương thức hủy theo các cách sau:

* Tạo một phương thức được chú thích bằng `@PreDestroy`.
* Implement phương thức `destroy` của `DisposableBean`.
* Sử dụng `@Bean(destroyMethod = "...")` trong lớp Configuration.

Khi sử dụng `AnnotationConfigApplicationContext`, hỗ trợ cho `@PostConstruct` và `@PreDestroy` được thêm vào tự động. Các annotation này được xử lý bởi `CommonAnnotationBeanPostProcessor`, được `AnnotationConfigApplicationContext` đăng ký tự động.

**Thứ tự gọi trong vòng đời:**

* **Khởi tạo:**
    1.  `BeanPostProcessor::postProcessBeforeInitialization` được gọi.
    2.  Phương thức `@PostConstruct` được gọi.
    3.  Phương thức `InitializingBean::afterPropertiesSet` được gọi.
    4.  Phương thức `@Bean(initMethod)` được gọi.
    5.  `BeanPostProcessor::postProcessAfterInitialization` được gọi.
* **Hủy (thường khi context đóng lại):**
    1.  Phương thức `@PreDestroy` được gọi.
    2.  Phương thức `DisposableBean::destroy` được gọi.
    3.  Phương thức `@Bean(destroyMethod)` được gọi.

#### Ví dụ Code

```java
// Một bean minh họa tất cả các phương thức vòng đời
public class LifecycleDemoBean implements InitializingBean, DisposableBean {

    public LifecycleDemoBean() {
        System.out.println("1. Constructor");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("2. @PostConstruct method");
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("3. InitializingBean's afterPropertiesSet()");
    }
    
    public void customInit() {
        System.out.println("4. Custom initMethod");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("5. @PreDestroy method");
    }

    @Override
    public void destroy() {
        System.out.println("6. DisposableBean's destroy()");
    }
    
    public void customDestroy() {
        System.out.println("7. Custom destroyMethod");
    }
}

@Configuration
public class AppConfig {
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public LifecycleDemoBean lifecycleDemoBean() {
        return new LifecycleDemoBean();
    }
}
```

### Câu hỏi 15: Component-scanning làm gì?

Component Scanning là một quá trình trong đó Spring quét Classpath để tìm kiếm các lớp được chú thích bằng các annotation stereotype (ví dụ: `@Component`, `@Repository`, `@Service`, `@Controller`) và dựa trên đó tạo ra các định nghĩa bean.

Bạn có thể kích hoạt quét component đơn giản trong gói của lớp Configuration và tất cả các gói con của nó. Bạn cũng có thể thiết lập các quy tắc quét nâng cao hơn.

#### Ví dụ Code

```java
package com.example.app;

@Configuration
// Quét gói "com.example.app" và tất cả các gói con của nó để tìm các component
@ComponentScan(basePackages = "com.example.app") 
public class AppConfig {
}

package com.example.app.services;

@Service // Stereotype annotation, sẽ được phát hiện bởi component scanning
public class MyService {
    public void doWork() {
        System.out.println("MyService is working.");
    }
}

package com.example.app.repositories;

@Repository // Stereotype annotation
public class MyRepository {
    // ...
}
```

### Câu hỏi 16: Hành vi của annotation @Autowired đối với field injection, constructor injection và method injection là gì?

`@Autowired` là một annotation được xử lý bởi `AutowiredAnnotationBeanPostProcessor`. Nó có thể được đặt trên constructor, trường, phương thức setter hoặc phương thức config của lớp. Việc sử dụng annotation này cho phép Spring tự động giải quyết các phụ thuộc, chủ yếu dựa trên kiểu dữ liệu. `@Autowired` có một thuộc tính `required` (mặc định là `true`) để cho Spring biết liệu một phụ thuộc có bắt buộc hay không.

**Các bước giải quyết phụ thuộc của `@Autowired`:**

1.  Khớp chính xác theo kiểu. Nếu chỉ tìm thấy một bean, quá trình kết thúc.
2.  Nếu tìm thấy nhiều bean cùng kiểu, kiểm tra xem có bean nào chứa annotation `@Primary` không. Nếu có, tiêm bean `@Primary` và kết thúc.
3.  Nếu vẫn không có kết quả khớp duy nhất, kiểm tra xem có tồn tại `@Qualifier` cho trường không. Nếu có, sử dụng `@Qualifier` để tìm bean khớp.
4.  Nếu vẫn không có bean duy nhất, thu hẹp tìm kiếm bằng cách sử dụng tên của bean.
5.  Nếu vẫn không tìm thấy một bean duy nhất, một ngoại lệ sẽ được ném ra.

**Field Injection (Tiêm vào trường):**

* Các trường được tự động tiêm có thể có bất kỳ mức độ hiển thị nào (public, protected, private, package-private).
* Việc tiêm diễn ra sau khi Bean được tạo nhưng trước khi bất kỳ phương thức init nào được gọi.

<!-- end list -->

```java
@Component
public class MyComponent {
    @Autowired
    private MyDependency dependency;
}
```

**Constructor Injection (Tiêm vào hàm khởi tạo):**

* Nếu chỉ có một hàm khởi tạo trong lớp, không cần phải sử dụng `@Autowired`, Spring sẽ tự động sử dụng nó.
* Nếu lớp định nghĩa nhiều hàm khởi tạo, bạn bắt buộc phải sử dụng `@Autowired` để cho Spring biết nên sử dụng hàm khởi tạo nào.

<!-- end list -->

```java
@Component
public class MyComponent {
    private final MyDependency dependency;

    // @Autowired là tùy chọn ở đây vì chỉ có một constructor
    public MyComponent(MyDependency dependency) {
        this.dependency = dependency;
    }
}
```

**Method Injection (Tiêm vào phương thức):**

* Phương thức được `@Autowired` có thể có bất kỳ mức độ hiển thị nào và có thể chứa nhiều tham số.
* Mặc định, tất cả các tham số trong phương thức được `@Autowired` đều được coi là bắt buộc.

<!-- end list -->

```java
@Component
public class MyComponent {
    private MyDependency dependency;
    private AnotherDependency anotherDependency;

    @Autowired
    public void configure(MyDependency dependency, AnotherDependency anotherDependency) {
        this.dependency = dependency;
        this.anotherDependency = anotherDependency;
    }
}
```

### Câu hỏi 17: Bạn phải làm gì nếu muốn tiêm một cái gì đó vào một trường private? Điều này ảnh hưởng đến việc kiểm thử như thế nào?

Việc tiêm một phụ thuộc vào một trường private có thể được thực hiện bằng annotation `@Autowired`. Việc tiêm một thuộc tính vào một trường private có thể được thực hiện bằng annotation `@Value`.

Vì trường private không thể được truy cập từ bên ngoài lớp, để giải quyết vấn đề này khi viết Unit Test, bạn có thể sử dụng các giải pháp sau:

* Sử dụng `SpringRunner` với `ContextConfiguration` và `@MockBean`.
* Sử dụng `ReflectionTestUtils` để sửa đổi các trường private.
* Sử dụng `MockitoJUnitRunner` để tiêm các mock.
* Sử dụng `@TestPropertySource` để tiêm các thuộc tính kiểm thử vào các trường private.

#### Ví dụ Code (Sử dụng ReflectionTestUtils trong test)

```java
// Lớp cần được test
@Component
public class ReportService {
    @Autowired
    private ReportGenerator reportGenerator;

    public String generateDailyReport() {
        return "Daily Report: " + reportGenerator.generate();
    }
}

// Lớp test
public class ReportServiceTest {
    private ReportService reportService;
    private ReportGenerator mockGenerator;

    @BeforeEach
    void setUp() {
        reportService = new ReportService();
        mockGenerator = Mockito.mock(ReportGenerator.class);
        
        // Sử dụng ReflectionTestUtils để tiêm mock vào trường private
        ReflectionTestUtils.setField(reportService, "reportGenerator", mockGenerator);
    }

    @Test
    void testGenerateDailyReport() {
        Mockito.when(mockGenerator.generate()).thenReturn("Data from Mock");
        
        String result = reportService.generateDailyReport();
        
        Assertions.assertEquals("Daily Report: Data from Mock", result);
    }
}
```

### Câu hỏi 18: Annotation @Qualifier bổ sung cho việc sử dụng @Autowired như thế nào?

Annotation `@Qualifier` cung cấp cho bạn quyền kiểm soát bổ sung về việc bean nào sẽ được tiêm khi có nhiều bean cùng loại được tìm thấy. Bằng cách thêm thông tin bổ sung về bean bạn muốn tiêm, `@Qualifier` giải quyết các vấn đề với `NoUniqueBeanDefinitionException`.

Bạn có thể sử dụng `@Qualifier` theo ba cách:

1.  Tại điểm tiêm với tên của bean làm giá trị.
2.  Tại cả điểm tiêm và điểm định nghĩa bean.
3.  Định nghĩa một Annotation Qualifier tùy chỉnh.

#### Ví dụ Code

```java
// Interface chung
public interface MessageService {
    void sendMessage(String message);
}

// Implementation 1
@Component("emailService")
public class EmailServiceImpl implements MessageService {
    @Override
    public void sendMessage(String message) { /* Gửi email */ }
}

// Implementation 2
@Component("smsService")
public class SmsServiceImpl implements MessageService {
    @Override
    public void sendMessage(String message) { /* Gửi SMS */ }
}

// Lớp sử dụng dependency
@Component
public class NotificationManager {
    private final MessageService messageService;

    // Sử dụng @Qualifier để chỉ định bean "smsService" cần được tiêm
    @Autowired
    public NotificationManager(@Qualifier("smsService") MessageService messageService) {
        this.messageService = messageService;
    }
}
```

### Câu hỏi 19: Proxy object là gì và hai loại proxy khác nhau mà Spring có thể tạo ra là gì?

Một **Proxy Object** là một đối tượng thêm logic bổ sung lên trên một đối tượng đang được ủy quyền (proxied) mà không cần phải sửa đổi mã của đối tượng gốc. Đối tượng proxy có các phương thức public giống như đối tượng được ủy quyền và nên khó phân biệt nhất có thể với đối tượng được ủy quyền. Khi một phương thức được gọi trên Proxy Object, mã bổ sung thường được gọi trước và sau khi mã từ đối tượng được ủy quyền được gọi bởi Proxy Object.

Spring Framework hỗ trợ hai loại proxy:

1.  **JDK Dynamic Proxy**: Được sử dụng theo mặc định nếu đối tượng mục tiêu implement một interface.
2.  **CGLIB Proxy**: Được sử dụng khi đối tượng mục tiêu không implement bất kỳ interface nào.

**Hạn chế:**

* **JDK Dynamic Proxy**: Yêu cầu đối tượng proxy phải implement interface, chỉ các phương thức của interface mới được ủy quyền, và không hỗ trợ tự gọi (self-invocation).
* **CGLIB Proxy**: Không hoạt động với các lớp `final` hoặc các phương thức `final`, và cũng không hỗ trợ tự gọi.

**Ưu điểm của Proxy:**

* Khả năng thay đổi hành vi của các bean hiện có mà không thay đổi mã gốc.
* Tách biệt các mối quan tâm (logging, transactions, security, ...).

**Nhược điểm của Proxy:**

* Có thể làm cho mã khó gỡ lỗi.
* Có thể gây ra vấn đề về hiệu suất nếu mã proxy sử dụng I/O.
* Có thể gây ra kết quả không mong đợi với toán tử `==` vì Proxy Object và Proxied Object là hai đối tượng khác nhau.

#### Ví dụ Code (Sử dụng AOP để minh họa Proxy)

Đây là một ví dụ về cách Spring sử dụng proxy để thêm hành vi logging vào một service.

```java
// Aspect để thêm logging
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* com.example.MyService.doWork(..))")
    public void logBefore() {
        System.out.println("Proxy: Logging before method execution...");
    }
}

// Service sẽ được ủy quyền (proxied)
@Service
public class MyService {
    public void doWork() {
        System.out.println("MyService: Doing actual work.");
    }
}

@Configuration
@ComponentScan
@EnableAspectJAutoProxy // Kích hoạt hỗ trợ AOP, sẽ tạo proxy
public class AppConfig {
}
```

Khi `myService.doWork()` được gọi, Spring sẽ thực sự gọi một proxy, proxy này sẽ in ra thông điệp log trước khi gọi phương thức `doWork()` thực sự trên đối tượng `MyService` gốc.

### Câu hỏi 20: Ưu điểm của Java Config là gì? Hạn chế là gì?

**Ưu điểm của Java Config so với XML Config**:

* Phản hồi tại thời gian biên dịch (Compile-Time Feedback) nhờ kiểm tra kiểu.
* Các công cụ tái cấu trúc (Refactoring Tools) cho Java hoạt động ngay lập tức với Java Config mà không cần plugin đặc biệt.

**Ưu điểm của Java Config so với Annotation Based Config**:

* Tách biệt các mối quan tâm - cấu hình bean được tách biệt khỏi việc triển khai bean.
* Không phụ thuộc vào công nghệ - các bean có thể không phụ thuộc vào việc triển khai IoC/DI cụ thể, giúp dễ dàng chuyển đổi công nghệ.
* Khả năng tích hợp Spring với các thư viện bên ngoài.
* Vị trí danh sách bean tập trung hơn.

**Hạn chế của Java Config**:

* Lớp cấu hình không thể là `final`.
* Các phương thức của lớp cấu hình không thể là `final`.
* Tất cả các Bean phải được liệt kê, đối với các ứng dụng lớn, đây có thể là một thách thức so với Component Scanning.

#### Ví dụ Code (Tích hợp thư viện bên ngoài)

Ví dụ này cho thấy cách sử dụng Java Config để tạo một bean cho thư viện `Gson` của Google, một thư viện mà bạn không thể thêm `@Component` vào mã nguồn của nó.

```java
import com.google.gson.Gson;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ExternalLibraryConfig {

    // Tạo một bean cho một đối tượng từ thư viện bên ngoài
    @Bean
    public Gson gson() {
        // Cung cấp một instance của Gson cho toàn bộ ứng dụng
        return new Gson();
    }
}

@Service
public class JsonProcessor {
    private final Gson gson;

    @Autowired
    public JsonProcessor(Gson gson) {
        this.gson = gson;
    }

    public String toJson(Object object) {
        return gson.toJson(object);
    }
}
```

### Câu hỏi 21: Annotation @Bean làm gì?

Annotation `@Bean` được sử dụng trong một lớp `@Configuration` để thông báo cho Spring rằng instance của lớp được trả về bởi phương thức được chú thích `@Bean` sẽ là một bean được quản lý bởi Spring.

`@Bean` cũng cho phép bạn:

* Chỉ định `initMethod` - sẽ được gọi sau khi instance được tạo và lắp ráp.
* Chỉ định `destroyMethod` - sẽ được gọi khi bean bị loại bỏ (thường là khi context đang đóng lại).
* Chỉ định `name` cho bean - theo mặc định, tên bean được tự động tạo dựa trên tên phương thức, tuy nhiên điều này có thể được ghi đè.
* Chỉ định `alias` (bí danh) cho bean.
* Chỉ định liệu Bean có nên được sử dụng làm ứng cử viên để tiêm vào các bean khác hay không - mặc định là `true`.

#### Ví dụ Code

```java
// Một lớp dịch vụ đơn giản
public class MyService {
    public void init() {
        System.out.println("MyService: Đã khởi tạo.");
    }

    public void doWork() {
        System.out.println("MyService: Đang làm việc.");
    }
    
    public void destroy() {
        System.out.println("MyService: Đang bị hủy.");
    }
}

@Configuration
public class AppConfig {
    
    @Bean(name = "customServiceBean", initMethod = "init", destroyMethod = "destroy")
    public MyService myService() {
        return new MyService();
    }
}

// Cách sử dụng
public class MainApp {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        // Lấy bean bằng tên tùy chỉnh
        MyService service = context.getBean("customServiceBean", MyService.class);
        service.doWork();
        
        // Đóng context để gọi destroy method
        context.close();
    }
}
```

### Câu hỏi 22: ID bean mặc định là gì nếu bạn chỉ sử dụng @Bean? Làm thế nào bạn có thể ghi đè nó?

Khi sử dụng `@Bean` mà không chỉ định tên hoặc bí danh, ID bean mặc định sẽ được tạo dựa trên tên của phương thức được chú thích bằng `@Bean`.

Bạn có thể ghi đè hành vi này bằng cách chỉ định thuộc tính `name` cho bean.

#### Ví dụ Code

```java
@Configuration
public class AppConfig {

    // Bean ID mặc định sẽ là "springBean1"
    @Bean
    public SpringBean1 springBean1() {
        return new SpringBean1();
    }

    // Ghi đè bean ID thành "customBeanName"
    @Bean(name = "customBeanName")
    public SpringBean2 springBean2() {
        return new SpringBean2();
    }
    
    // Ghi đè bean ID và cung cấp nhiều bí danh
    @Bean(name = { "mainBean3", "thirdSpringBean" })
    public SpringBean3 springBean3() {
        return new SpringBean3();
    }
}
```

### Câu hỏi 23: Tại sao bạn không được phép chú thích một lớp final bằng @Configuration?

Một lớp được chú thích bằng `@Configuration` không thể là `final` vì Spring sẽ sử dụng CGLIB để tạo một proxy cho lớp `@Configuration` đó. CGLIB tạo ra một lớp con (subclass) cho mỗi lớp cần được ủy quyền (proxied). Tuy nhiên, vì một lớp `final` không thể có lớp con, CGLIB sẽ thất bại.

Đây cũng là lý do tại sao các phương thức `@Bean` không thể là `final`. Spring cần ghi đè các phương thức từ lớp cha để proxy hoạt động chính xác, nhưng một phương thức `final` không thể bị ghi đè.

Spring hỗ trợ các bean `Singleton` trong một lớp `@Configuration` bằng cách tạo một proxy CGLIB chặn các cuộc gọi đến phương thức. Trước khi phương thức được thực thi, proxy sẽ chặn cuộc gọi và kiểm tra xem instance của bean đã tồn tại hay chưa. Nếu đã tồn tại, proxy sẽ trả về instance hiện có. Nếu không, nó sẽ cho phép cuộc gọi đến phương thức gốc để tạo bean, sau đó lưu lại instance để sử dụng trong tương lai.

#### Ví dụ Code

Ví dụ dưới đây minh họa cách Spring đảm bảo tính singleton ngay cả khi một phương thức `@Bean` gọi một phương thức `@Bean` khác.

```java
@Configuration
public class AppConfig {

    @Bean
    public ServiceA serviceA() {
        System.out.println("Tạo ServiceA...");
        // Gọi phương thức serviceB() để lấy dependency.
        // Nhờ có proxy CGLIB, nó sẽ trả về cùng một instance của ServiceB
        // thay vì tạo một instance mới.
        return new ServiceA(serviceB());
    }
    
    @Bean
    public ServiceB serviceB() {
        System.out.println("Tạo ServiceB...");
        return new ServiceB();
    }
}

// Trong ứng dụng, khi context được khởi tạo, bạn sẽ thấy "Tạo ServiceB..." chỉ được in ra một lần,
// chứng tỏ serviceB() chỉ được thực thi một lần duy nhất.
```

### Câu hỏi 24: Làm thế nào để cấu hình profile? Các trường hợp sử dụng hữu ích của chúng là gì?

Bạn cấu hình Spring Profile bằng cách:

* Chỉ định bean nào thuộc về profile nào.
* Chỉ định profile nào đang hoạt động.

Bạn có thể chỉ định bean thuộc về một profile bằng các cách sau:

* Sử dụng `@Profile` ở cấp độ lớp `@Component` hoặc `@Configuration`.
* Sử dụng `@Profile` ở cấp độ phương thức `@Bean` của lớp `@Configuration`.

Nếu một Bean không có profile nào được chỉ định, nó sẽ được tạo trong mọi profile. Bạn có thể sử dụng toán tử `!` để chỉ định profile mà bean không nên được tạo.

Bạn có thể kích hoạt các profile bằng các cách sau:

* Lập trình bằng cách sử dụng `ConfigurableEnvironment`.
* Sử dụng thuộc tính `spring.profiles.active`.
* Ở cấp độ JUnit Test bằng cách sử dụng annotation `@ActiveProfiles`.
* Trong Spring Boot, bằng cách sử dụng `SpringApplicationBuilder` hoặc trong tệp `application.properties`/`yml`.

**Các trường hợp sử dụng hữu ích của Spring Profiles:**

* Thay đổi hành vi của hệ thống trong các môi trường khác nhau (ví dụ: dev, test, prod).
* Thay đổi hành vi của hệ thống cho các khách hàng khác nhau.
* Thay đổi tập hợp các Bean được sử dụng trong môi trường phát triển và trong quá trình kiểm thử.
* Bật/tắt các tính năng giám sát hoặc gỡ lỗi bổ sung.

#### Ví dụ Code

```java
// Interface chung cho nguồn dữ liệu
public interface DataSourceConfig {
    void setup();
}

@Configuration
public class AppConfig {
    
    // Bean này chỉ hoạt động khi profile "dev" được kích hoạt
    @Bean
    @Profile("dev")
    public DataSourceConfig devDataSource() {
        return new DevDataSource();
    }

    // Bean này chỉ hoạt động khi profile "prod" được kích hoạt
    @Bean
    @Profile("prod")
    public DataSourceConfig prodDataSource() {
        return new ProdDataSource();
    }
}

// Để kích hoạt profile "dev", bạn có thể chạy ứng dụng với tham số VM:
// -Dspring.profiles.active=dev
```

### Câu hỏi 25: Bạn có thể sử dụng @Bean cùng với @Profile không?

Có, annotation `@Bean` có thể được sử dụng cùng với `@Profile` bên trong một lớp được chú thích bằng `@Configuration`, đặt trên phương thức trả về instance của bean.

Nếu phương thức được chú thích bằng `@Bean` không có `@Profile`, thì bean đó sẽ tồn tại trong tất cả các profile.

#### Ví dụ Code

Ví dụ đã được trình bày trong Câu hỏi 24, nơi các phương thức `devDataSource()` và `prodDataSource()` được chú thích bằng cả `@Bean` và `@Profile`.

### Câu hỏi 26: Bạn có thể sử dụng @Component cùng với @Profile không?

Có, annotation `@Profile` có thể được sử dụng cùng với `@Component` trên một lớp đại diện cho một spring bean.

Nếu một lớp được chú thích bằng `@Component` không có `@Profile`, bean đó sẽ tồn tại trong tất cả các profile.

#### Ví dụ Code

```java
// Component này chỉ được tạo khi profile "dev" hoạt động
@Component
@Profile("dev")
public class DevelopmentNotifier implements Notifier {
    @Override
    public void send(String message) {
        System.out.println("DEV Notifier: " + message);
    }
}

// Component này chỉ được tạo khi profile "prod" hoạt động
@Component
@Profile("prod")
public class ProductionNotifier implements Notifier {
    @Override
    public void send(String message) {
        // Gửi email, SMS, v.v. trong môi trường production
    }
}
```

### Câu hỏi 27: Bạn có thể có bao nhiêu profile?

Spring Framework không quy định bất kỳ giới hạn rõ ràng nào về số lượng profile. Tuy nhiên, vì một số lớp trong Framework sử dụng mảng để lặp qua các profile, điều này áp đặt một giới hạn ngầm định bằng với số lượng phần tử tối đa trong một mảng mà bạn có thể có trong Java, đó là `Integer.MAX_VALUE - 1` (2,147,483,647).

### Câu hỏi 28: Làm thế nào để tiêm các giá trị vô hướng/chuỗi ký tự (scalar/literal) vào các Spring bean?

Để tiêm các giá trị vô hướng/chuỗi ký tự vào Spring Beans, bạn cần sử dụng annotation `@Value`. Annotation `@Value` có một trường `value` chấp nhận:

* Một giá trị đơn giản.
* Một tham chiếu thuộc tính (`${...}`).
* Một chuỗi SpEL (`#{...}`).

Bên trong `@Value`, bạn có thể chỉ định:

* Giá trị đơn giản: `@Value("John")`, `@Value("true")`.
* Tham chiếu đến một thuộc tính: `@Value("${app.department.id}")`.
* Thực hiện tính toán nội tuyến bằng SpEL: `@Value("#{'Wall Street'.toUpperCase()}")`.

#### Ví dụ Code

Giả sử có tệp `application.properties`:
`app.name=My App`

```java
@Component
public class AppDetails {
    // Tiêm một giá trị chuỗi ký tự cứng
    @Value("1.2.3")
    private String version;

    // Tiêm một giá trị từ tệp properties
    @Value("${app.name}")
    private String applicationName;
    
    // Tiêm một giá trị được tính toán bởi SpEL
    @Value("#{365 * 24}")
    private int hoursInYear;

    public void display() {
        System.out.println("Version: " + version);
        System.out.println("App Name: " + applicationName);
        System.out.println("Hours in Year: " + hoursInYear);
    }
}
```

### Câu hỏi 29: @Value được dùng để làm gì?

`@Value` được sử dụng cho:

* Thiết lập các giá trị đơn giản cho các trường, tham số phương thức, tham số hàm khởi tạo của Spring Bean.
* Tiêm các giá trị từ thuộc tính/môi trường.
* Tiêm kết quả của các biểu thức SpEL.
* Tiêm giá trị từ các Spring Bean khác.
* Tiêm giá trị vào các collection (mảng, list, set, map).
* Thiết lập các giá trị mặc định khi giá trị được tham chiếu bị thiếu.

#### Ví dụ Code

```java
@Component
public class AdvancedValueInjection {

    // Tiêm một danh sách các chuỗi ký tự
    @Value("#{{'one', 'two', 'three'}}")
    private List<String> stringList;
    
    // Tiêm một thuộc tính với giá trị mặc định là "unknown" nếu không tìm thấy
    @Value("${unknown.property:unknown}")
    private String propertyWithDefault;
    
    // Tiêm giá trị từ một bean khác (giả sử có bean tên "appDetails")
    @Value("#{appDetails.version}")
    private String appVersionFromAnotherBean;
}
```

### Câu hỏi 30: Spring Expression Language (SpEL) là gì?

Spring Expression Language (SpEL) là một ngôn ngữ biểu thức cho phép bạn truy vấn và thao tác các đồ thị đối tượng trong thời gian chạy. SpEL được sử dụng trong các sản phẩm khác nhau trong danh mục của Spring.

SpEL có thể được sử dụng độc lập hoặc trên các trường, tham số phương thức, đối số hàm khởi tạo thông qua annotation `@Value("#{}")`.

Các tính năng SpEL hỗ trợ bao gồm: biểu thức chuỗi ký tự, toán tử boolean và quan hệ, biểu thức chính quy, truy cập thuộc tính, mảng, danh sách và map, gọi phương thức, tham chiếu bean, toán tử ba ngôi, biến, và nhiều hơn nữa.

Mặc dù biểu thức SpEL thường được thông dịch trong thời gian chạy, Spring Framework 4.1 đã giới thiệu khả năng biên dịch các biểu thức để có hiệu suất tốt hơn. Trình biên dịch tạo ra một lớp Java thực sự thể hiện biểu thức, giúp đánh giá biểu thức nhanh hơn nhiều. Trình biên dịch bị tắt theo mặc định.

#### Ví dụ Code

```java
@Component("inventory")
public class Inventory {
    private Map<String, Integer> items = new HashMap<>();
    
    public Inventory() {
        items.put("Laptop", 10);
        items.put("Mouse", 50);
    }
    
    public int getStock(String itemName) {
        return items.getOrDefault(itemName, 0);
    }
}

@Component
public class StoreStatus {
    
    // Sử dụng SpEL để gọi phương thức trên một bean khác
    @Value("#{inventory.getStock('Laptop')}")
    private int laptopStock;
    
    // Sử dụng SpEL cho các phép toán logic và quan hệ
    @Value("#{inventory.getStock('Mouse') > 20}")
    private boolean mouseStockSufficient;

    // Sử dụng SpEL để thao tác chuỗi
    @Value("#{'Store Status'.toUpperCase()}")
    private String reportTitle;

    public void printStatus() {
        System.out.println(reportTitle);
        System.out.println("Laptop Stock: " + laptopStock);
        System.out.println("Mouse Stock is Sufficient: " + mouseStockSufficient);
    }
}
```

### Câu hỏi 31: Khái niệm trừu tượng Environment trong Spring là gì?

Khái niệm trừu tượng `Environment` là một phần của Spring Container, mô hình hóa hai khía cạnh chính của môi trường ứng dụng: **Profiles** và **Properties**.

Ở cấp độ mã nguồn, `Environment` được đại diện bởi các lớp implement interface `Environment`. Interface này cho phép bạn giải quyết các thuộc tính (properties) và liệt kê các profile. Vai trò của `Environment` trong bối cảnh profile là xác định profile nào hiện đang hoạt động. Trong bối cảnh properties, vai trò của nó là cung cấp một dịch vụ tiện lợi, được tiêu chuẩn hóa và chung chung cho phép giải quyết các thuộc tính và cấu hình các nguồn thuộc tính (property sources).

Các thuộc tính có thể đến từ nhiều nguồn khác nhau như:

* Tệp Properties
* Thuộc tính hệ thống JVM
* Biến môi trường hệ thống
* JNDI
* Tham số Servlet Config
* Tham số Servlet Context

Bạn có thể thêm các tệp thuộc tính bổ sung làm nguồn thuộc tính bằng cách sử dụng annotation `@PropertySource`.

#### Ví dụ Code

Bạn có thể tiêm `Environment` vào bean của mình để truy cập các thuộc tính và thông tin profile một cách linh hoạt.

```java
@Component
public class EnvironmentReader {

    private final Environment environment;

    @Autowired
    public EnvironmentReader(Environment environment) {
        this.environment = environment;
    }

    public void displayAppProperties() {
        // Lấy một thuộc tính từ environment
        String appName = environment.getProperty("app.name");
        System.out.println("Application Name: " + appName);

        // Kiểm tra xem profile "dev" có đang hoạt động không
        if (environment.acceptsProfiles(Profiles.of("dev"))) {
            System.out.println("Chế độ Development đang được kích hoạt.");
            String devDbUrl = environment.getProperty("dev.db.url");
            System.out.println("Dev DB URL: " + devDbUrl);
        }
    }
}
```

### Câu hỏi 32: Các thuộc tính trong môi trường có thể đến từ đâu?

Các nguồn thuộc tính (Property Sources) trong một ứng dụng Spring thay đổi tùy thuộc vào loại ứng dụng đang được thực thi.

**Đối với ứng dụng Spring Framework độc lập (Standalone):**

* Tệp Properties
* Thuộc tính hệ thống JVM
* Biến môi trường hệ thống

**Đối với ứng dụng Spring Framework trong Servlet Container:**

* Tệp Properties
* Thuộc tính hệ thống JVM
* Biến môi trường hệ thống
* JNDI
* Tham số khởi tạo `ServletConfig`
* Tham số khởi tạo `ServletContext`

**Đối với ứng dụng Spring Boot (thứ tự ưu tiên từ thấp đến cao):**
Spring Boot mở rộng danh sách này một cách đáng kể, cung cấp một hệ thống phân cấp rất linh hoạt. Dưới đây là một số nguồn phổ biến, được sắp xếp theo thứ tự ưu tiên (các mục ở sau sẽ ghi đè các mục ở trước):

1.  Các thuộc tính mặc định (`SpringApplication.setDefaultProperties`).
2.  Annotation `@PropertySource` trên các lớp `@Configuration`.
3.  Tệp `application.properties` (hoặc `.yml`) bên trong JAR.
4.  Tệp `application-{profile}.properties` (hoặc `.yml`) bên trong JAR.
5.  Tệp `application.properties` (hoặc `.yml`) bên ngoài JAR.
6.  Tệp `application-{profile}.properties` (hoặc `.yml`) bên ngoài JAR.
7.  Thuộc tính hệ thống JVM (`System.getProperties()`).
8.  Các đối số dòng lệnh (command line arguments).
9.  Annotation `@TestPropertySource` trong các bài test.

### Câu hỏi 33: Bạn có thể tham chiếu những gì bằng cách sử dụng SpEL?

Bạn có thể tham chiếu những thứ sau đây bằng SpEL (Spring Expression Language):

* **Trường tĩnh từ một lớp:** `T(com.example.Person).DEFAULT_NAME`
* **Phương thức tĩnh từ một lớp:** `T(com.example.Person).getDefaultName()`
* **Thuộc tính của một Spring Bean:** `@person.name`
* **Phương thức của một Spring Bean:** `@person.getName()`
* **Biến SpEL:** `#personName`
* **Thuộc tính của đối tượng trên một biến SpEL:** `#person.name`
* **Phương thức của đối tượng trên một biến SpEL:** `#person.getName()`
* **Thuộc tính từ môi trường ứng dụng Spring:** `environment['app.file.property']`
* **Thuộc tính hệ thống:** `systemProperties['app.vm.property']`
* **Biến môi trường hệ thống:** `systemEnvironment['JAVA_HOME']`

#### Ví dụ Code

```java
// Giả sử có một bean tên là "calculator"
@Component("calculator")
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}

@Component
public class SpelDemo {

    // Gọi phương thức trên một bean khác
    @Value("#{calculator.add(10, 20)}")
    private int sum;
    
    // Truy cập một thuộc tính hệ thống (ví dụ: java.version)
    @Value("#{systemProperties['java.version']}")
    private String javaVersion;

    // Sử dụng toán tử và biểu thức
    @Value("#{T(java.lang.Math).PI * 2 * 5}")
    private double circleCircumference;

    public void showValues() {
        System.out.println("Sum from calculator bean: " + sum);
        System.out.println("Java Version: " + javaVersion);
        System.out.println("Circumference of a circle with radius 5: " + circleCircumference);
    }
}
```

### Câu hỏi 34: Sự khác biệt giữa $ và \# trong các biểu thức @Value là gì?

Annotation `@Value` hỗ trợ hai loại biểu thức:

1.  **Biểu thức bắt đầu bằng `$` (dấu đô la):** Được sử dụng để tham chiếu đến một **thuộc tính** trong `Environment` của Spring. Spring sẽ tìm kiếm thuộc tính có tên tương ứng trong tất cả các nguồn thuộc tính đã được cấu hình (ví dụ: `application.properties`, biến môi trường, v.v.).

2.  **Biểu thức bắt đầu bằng `#` (dấu thăng):** Đây là các biểu thức **SpEL (Spring Expression Language)**. Chúng được phân tích cú pháp và đánh giá bởi SpEL, cho phép bạn thực hiện các thao tác phức tạp hơn như gọi phương thức, thực hiện phép toán, truy cập các bean khác, v.v.

#### Ví dụ Code

```java
// Giả sử có tệp application.properties:
// server.port=8080

@Component
public class ValueExpressionDemo {
    
    // Sử dụng '$' để lấy giá trị từ tệp properties
    // Spring sẽ tìm thuộc tính 'server.port' trong môi trường
    @Value("${server.port}")
    private int serverPort;
    
    // Sử dụng '#' để thực thi một biểu thức SpEL
    // Ở đây, chúng ta đang nhân 20 với 5
    @Value("#{20 * 5}")
    private int calculatedValue;
    
    // Bạn cũng có thể kết hợp cả hai
    // Lấy giá trị port, chuyển nó thành số nguyên và cộng thêm 1
    @Value("#{'${server.port}' + 1}")
    private int adminPort;
    
    public void display() {
        System.out.println("Server Port (from property): " + serverPort);
        System.out.println("Calculated Value (from SpEL): " + calculatedValue);
        System.out.println("Admin Port (from combination): " + adminPort);
    }
}
```