## 1\. Bạn có sử dụng Spring trong unit test không?

Thông thường, Spring Framework **không được sử dụng** trong các bài kiểm thử đơn vị (unit test). Unit test được thiết kế để kiểm thử một đơn vị chức năng (thường là một lớp) một cách cô lập. Điều này có nghĩa là các thành phần phụ thuộc (collaborators) của lớp đó cần được giả lập (mock), và không cần khởi động một môi trường hay một IoC/DI container nào.

Tuy nhiên, Spring cung cấp một số lớp tiện ích để hỗ trợ việc viết unit test trong các trường hợp đặc biệt:

* **`ReflectionTestUtils`**: Dùng để thiết lập các trường (field) private, hữu ích khi bạn cần tiêm (inject) các mock một cách thủ công mà không có container.
* **Các lớp Mock**: Spring cung cấp các đối tượng mock cho Servlet API (`MockHttpServletRequest`), Environment (`MockEnvironment`), và nhiều thứ khác để bạn có thể kiểm thử các thành phần phụ thuộc vào chúng mà không cần môi trường thực.

**Ví dụ Code: Unit Test không dùng Spring**

Đây là một ví dụ về unit test cho một `CalculatorService` sử dụng JUnit và Mockito. Hoàn toàn không cần khởi động Spring context.

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.when;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

// Service phụ thuộc để giả lập
interface TaxProvider {
    double getVatRate();
}

// Lớp cần được kiểm thử (Class Under Test)
class CalculatorService {
    private final TaxProvider taxProvider;

    public CalculatorService(TaxProvider taxProvider) {
        this.taxProvider = taxProvider;
    }

    public double calculatePriceWithVat(double price) {
        double vatRate = taxProvider.getVatRate();
        return price * (1 + vatRate);
    }
}

// Sử dụng Mockito runner để xử lý các annotation @Mock và @InjectMocks
@ExtendWith(MockitoExtension.class)
public class CalculatorServiceTest {

    @Mock
    private TaxProvider mockTaxProvider; // Tạo một mock cho dependency

    @InjectMocks
    private CalculatorService calculatorService; // Tạo instance của lớp cần test và inject mock vào đó

    @Test
    void testCalculatePriceWithVat() {
        // 1. Arrange (Sắp đặt): Định nghĩa hành vi của mock
        when(mockTaxProvider.getVatRate()).thenReturn(0.1); // Giả sử thuế suất là 10%

        // 2. Act (Hành động): Gọi phương thức cần test
        double result = calculatorService.calculatePriceWithVat(100.0);

        // 3. Assert (Xác nhận): Kiểm tra kết quả
        assertEquals(110.0, result);
    }
}
```

-----

## 2\. Loại test nào thường sử dụng Spring?

**Kiểm thử tích hợp (Integration Tests)** là loại test thường sử dụng Spring nhất.

Lý do là vì ở cấp độ kiểm thử tích hợp, mục tiêu của chúng ta là kiểm tra sự phối hợp của nhiều thành phần với nhau trong một môi trường gần giống với môi trường production. Để làm được điều này, chúng ta cần một IoC/DI container để khởi tạo, liên kết (wire) và quản lý vòng đời của các đối tượng (bean). Spring Framework làm rất tốt việc này.

Spring cung cấp sự hỗ trợ tuyệt vời cho kiểm thử tích hợp, với các mục tiêu chính:

* Quản lý và cache Spring IoC container giữa các bài test để tăng tốc độ thực thi.
* Hỗ trợ tiêm phụ thuộc (Dependency Injection) vào các lớp test bằng các annotation như `@Autowired`.
* Quản lý giao dịch (transaction) để đảm bảo các bài test không ảnh hưởng đến dữ liệu của nhau, thường bằng cách tự động rollback sau mỗi test.

**Ví dụ Code: Integration Test với Spring Boot**

Ví dụ này kiểm thử `UserService`, vốn phụ thuộc vào `UserRepository` để tương tác với cơ sở dữ liệu. `@SpringBootTest` sẽ khởi động Spring context để `UserService` và `UserRepository` được tạo và tiêm vào nhau.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

// Giả sử có các lớp User, UserRepository và UserService đã được định nghĩa
/*
@Service
public class UserService {
    private final UserRepository userRepository;
    // ... constructor
    public User findUser(String username) { ... }
}
*/

// @SpringBootTest sẽ tìm cấu hình @SpringBootApplication và khởi động context
@SpringBootTest
public class UserServiceIntegrationTest {

    @Autowired
    private UserService userService; // Spring sẽ tiêm một instance thực của UserService

    @Autowired
    private UserRepository userRepository; // Spring cũng tiêm một instance thực của UserRepository

    @Test
    void whenUserExists_thenFindUserShouldReturnUser() {
        // Arrange: Chuẩn bị dữ liệu test
        User testUser = new User("testuser", "password");
        userRepository.save(testUser);

        // Act: Gọi logic nghiệp vụ
        User foundUser = userService.findUser("testuser");

        // Assert: Kiểm tra kết quả
        assertEquals("testuser", foundUser.getUsername());
    }
}
```

-----

## 3\. Làm thế nào để tạo một application context dùng chung trong một integration test JUnit?

Việc chia sẻ một `ApplicationContext` giữa các bài test tích hợp là một kỹ thuật quan trọng để tăng tốc độ thực thi, vì việc khởi tạo context tốn nhiều thời gian.

Spring Test Context Framework sẽ tự động **cache và tái sử dụng** các context theo mặc định. Một context sẽ được dùng chung giữa các bài test miễn là cấu hình của chúng hoàn toàn giống nhau. Cấu hình này bao gồm các thuộc tính trong `@ContextConfiguration` (như `classes`, `locations`), `@ActiveProfiles`, `@TestPropertySource`, và các annotation khác.

Để chia sẻ định nghĩa context một cách nhất quán giữa các lớp test, bạn có thể:

* Tạo một **lớp test cơ sở (base class)** chứa các annotation cấu hình chung (`@SpringBootTest`, `@ContextConfiguration`, `@ActiveProfiles`). Các lớp test khác sẽ kế thừa từ lớp này.
* Tạo một **annotation tùy chỉnh (custom annotation)** bao gồm các annotation cấu hình và sử dụng nó trên các lớp test.

Để buộc Spring phải tạo một context mới và không dùng lại từ cache, bạn có thể sử dụng annotation `@DirtiesContext`.

**Ví dụ Code: Sử dụng Base Class để chia sẻ Context**

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.beans.factory.annotation.Autowired;
import org.junit.jupiter.api.Test;

// Lớp cơ sở chứa cấu hình chung cho tất cả các integration test
@SpringBootTest
@ActiveProfiles("test") // Tất cả các test kế thừa sẽ chạy với profile "test"
public abstract class BaseIntegrationTest {
    // Lớp này có thể trống hoặc chứa các tiện ích chung
}

// ----- Test Class 1 -----
public class ProductServiceIntegrationTest extends BaseIntegrationTest {
    @Autowired
    private ProductService productService;

    @Test
    void testSomethingWithProducts() {
        // ...
    }
}

// ----- Test Class 2 -----
public class OrderServiceIntegrationTest extends BaseIntegrationTest {
    @Autowired
    private OrderService orderService;

    @Test
    void testSomethingWithOrders() {
        // ...
    }
}
```

Trong ví dụ này, Spring sẽ chỉ tạo `ApplicationContext` một lần và tái sử dụng nó cho cả `ProductServiceIntegrationTest` và `OrderServiceIntegrationTest`.

-----

## 4\. Khi nào và ở đâu bạn sử dụng @Transactional trong testing?

Annotation `@Transactional` được sử dụng trong các bài test tích hợp khi bạn cần thực thi các đoạn mã có khả năng thay đổi trạng thái của một tài nguyên giao dịch, điển hình nhất là cơ sở dữ liệu.

Mục đích chính của nó trong testing là để đảm bảo tính cô lập của các bài test. Theo mặc định, sau khi một phương thức test được chú thích bằng `@Transactional` kết thúc, giao dịch sẽ được **rollback (hoàn tác)**. Điều này có nghĩa là mọi thay đổi đối với cơ sở dữ liệu (thêm, sửa, xóa) trong quá trình test sẽ bị hủy bỏ, trả lại cơ sở dữ liệu về trạng thái ban đầu trước khi bài test tiếp theo được chạy.

Bạn có thể sử dụng `@Transactional` ở hai cấp độ:

* Trên một lớp test: Tất cả các phương thức test trong lớp đó sẽ được chạy trong một giao dịch.
* Trên một phương thức test: Chỉ phương thức test đó sẽ được chạy trong một giao dịch.

Để kiểm soát hành vi này, bạn có thể sử dụng các annotation bổ sung như `@Commit` (để buộc commit giao dịch) hoặc `@Rollback(false)`.

**Ví dụ Code: Sử dụng @Transactional trong Test**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertTrue;

@SpringBootTest
public class UserRepositoryIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @Transactional // Bắt đầu một giao dịch trước khi test, và rollback sau khi test kết thúc
    void whenSaveUser_thenUserShouldBePersisted() {
        // Act
        User newUser = new User("testuser", "test@example.com");
        userRepository.save(newUser);

        // Assert
        // Chúng ta có thể kiểm tra xem người dùng có tồn tại trong DB trong phạm vi giao dịch này không
        assertTrue(userRepository.findByUsername("testuser").isPresent());
    }

    // Sau khi phương thức trên kết thúc, người dùng "testuser" sẽ bị xóa khỏi DB
    // nhờ cơ chế rollback của @Transactional.
}
```

-----

## 5\. Các mock framework như Mockito hay EasyMock được sử dụng như thế nào?

Các mock framework như Mockito được sử dụng để tạo ra các **đối tượng giả lập (mock objects)**. Mục đích chính là để **cô lập lớp đang được kiểm thử (class under test)** khỏi các thành phần phụ thuộc (collaborators) của nó, chủ yếu trong các bài unit test.

Một đối tượng mock có thể "giả vờ" là một đối tượng thật và trả về các kết quả được định nghĩa trước khi một phương thức của nó được gọi. Điều này cho phép bạn kiểm tra logic của một lớp mà không cần đến sự hoạt động thực sự của các lớp phụ thuộc. Ngoài ra, mock framework còn cho phép bạn **xác minh (verify)** xem các phương thức mong đợi trên đối tượng mock có được gọi với các tham số chính xác hay không.

Trong các bài **integration test**, mock cũng có thể được sử dụng để giả lập một phần của hệ thống, ví dụ như một dịch vụ bên ngoài (external API) hoặc một thành phần chưa được triển khai.

**Ví dụ Code: Sử dụng Mockito**

Đây là một ví dụ về việc sử dụng Mockito để test một `ReportService` phụ thuộc vào `UserDataRepository`.

```java
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import java.util.Arrays;

// Các lớp ví dụ
interface UserDataRepository {
    int[] findUserActivity(int userId);
}

class ReportService {
    private final UserDataRepository repository;
    public ReportService(UserDataRepository repository) { this.repository = repository; }

    public String generateActivityReport(int userId) {
        int[] activities = repository.findUserActivity(userId);
        if (activities.length == 0) return "Không có hoạt động";
        long sum = Arrays.stream(activities).sum();
        return "Tổng hoạt động: " + sum;
    }
}

@ExtendWith(MockitoExtension.class)
public class ReportServiceTest {
    @Mock
    private UserDataRepository mockRepository; // Tạo mock

    @InjectMocks
    private ReportService reportService; // Inject mock vào service

    @Test
    void testReportGeneration() {
        // 1. Stubbing: Định nghĩa hành vi của mock
        // Khi phương thức findUserActivity được gọi với userId là 1, trả về một mảng
        when(mockRepository.findUserActivity(1)).thenReturn(new int[]{10, 20, 30});

        // 2. Gọi phương thức cần test
        String report = reportService.generateActivityReport(1);

        // 3. Xác nhận kết quả
        assertEquals("Tổng hoạt động: 60", report);

        // 4. Verification: Kiểm tra xem phương thức trên mock có được gọi đúng hay không
        verify(mockRepository).findUserActivity(1);
    }
}
```

-----

## 6\. @ContextConfiguration được sử dụng như thế nào?

`@ContextConfiguration` là một annotation cốt lõi trong Spring Test Framework, được sử dụng trên một lớp test tích hợp để chỉ định cách tải (load) và cấu hình một `ApplicationContext` cho bài test đó.

Đây là cách "truyền thống" để thiết lập một integration test trước khi có sự đơn giản hóa của Spring Boot và `@SpringBootTest`.

`@ContextConfiguration` có hai cách tiếp cận chính:

* **Dựa trên lớp chú thích (Annotated Classes)**: Bạn cung cấp một hoặc nhiều lớp được chú thích bằng `@Configuration` cho thuộc tính `classes`. Spring sẽ sử dụng các lớp này để xây dựng context.
* **Dựa trên XML**: Bạn cung cấp đường dẫn đến một hoặc nhiều tệp XML cấu hình Spring cho thuộc tính `locations`.

**Ví dụ Code: Sử dụng @ContextConfiguration với lớp Configuration**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import static org.junit.jupiter.api.Assertions.assertNotNull;

// Một dịch vụ đơn giản
class MyService {
    public String getMessage() {
        return "Hello from MyService!";
    }
}

// Một lớp cấu hình cho test
@Configuration
class TestConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}

// Lớp test
@ExtendWith(SpringExtension.class) // Tương đương @RunWith(SpringRunner.class) trong JUnit 4
// Chỉ định cách tải context bằng cách sử dụng lớp TestConfig
@ContextConfiguration(classes = TestConfig.class)
public class MyServiceContextTest {

    @Autowired
    private MyService myService;

    @Test
    void testServiceIsNotNull() {
        assertNotNull(myService, "MyService nên được tiêm vào.");
        System.out.println(myService.getMessage());
    }
}
```

-----

## 7\. Spring Boot đơn giản hóa việc viết test như thế nào?

Spring Boot đơn giản hóa đáng kể việc viết test bằng cách cung cấp các tính năng tự động cấu hình và các công cụ tiện lợi.

Các cách chính mà Spring Boot giúp đơn giản hóa việc viết test bao gồm:

* **Cung cấp starter `spring-boot-starter-test`**: Starter này gộp sẵn các thư viện kiểm thử phổ biến nhất như JUnit, Spring Test, Spring Boot Test, AssertJ, Hamcrest, Mockito, và JSONassert.
* **Annotation `@SpringBootTest`**: Đây là một annotation "tất cả trong một" để chạy các bài test tích hợp. Nó tự động tìm kiếm cấu hình chính của bạn, tạo `ApplicationContext`, và cung cấp các tính năng của Spring Boot.
* **Tự động cấu hình cho test (Test Auto-Configuration)**: Dựa trên các phụ thuộc trong classpath, Spring Boot có thể tự động cấu hình các thành phần cần thiết cho test, ví dụ như một cơ sở dữ liệu trong bộ nhớ (in-memory database).
* **Các "lát cắt" test (Test Slices)**: Cung cấp các annotation chuyên dụng để chỉ kiểm thử một phần ("lát cắt") của ứng dụng, giúp test chạy nhanh hơn. Ví dụ:
    * `@WebMvcTest`: Chỉ kiểm thử tầng web (controller), không tải toàn bộ context.
    * `@DataJpaTest`: Chỉ kiểm thử tầng persistence của JPA.
    * `@JsonTest`: Chỉ kiểm thử việc tuần tự hóa/giải tuần tự hóa JSON.
    * `@RestClientTest`: Chỉ kiểm thử các REST client.
* **Tích hợp Mockito**: Cung cấp các annotation `@MockBean` và `@SpyBean` để dễ dàng tạo và tiêm các mock/spy của Mockito vào Spring context.

**Ví dụ Code: Sử dụng @WebMvcTest**

Ví dụ này chỉ kiểm thử `GreetingController` mà không cần khởi động toàn bộ ứng dụng. `GreetingService` được giả lập bằng `@MockBean`.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

// Lớp ví dụ
/*
@Service public class GreetingService { ... }
@RestController public class GreetingController { ... }
*/

// @WebMvcTest chỉ tập trung vào lớp GreetingController
@WebMvcTest(GreetingController.class)
public class GreetingControllerTest {

    @Autowired
    private MockMvc mockMvc; // MockMvc được Spring Boot tự động cấu hình

    @MockBean
    private GreetingService greetingService; // Tạo một mock cho GreetingService và đưa vào context

    @Test
    void greetingShouldReturnMessageFromService() throws Exception {
        // Arrange: Định nghĩa hành vi của mock
        when(greetingService.greet()).thenReturn("Xin chào, Mock!");

        // Act & Assert: Thực hiện request và kiểm tra kết quả
        this.mockMvc.perform(get("/greeting"))
            .andExpect(status().isOk())
            .andExpect(content().string("Xin chào, Mock!"));
    }
}
```

-----

## 8\. @SpringBootTest làm gì? Nó tương tác với @SpringBootApplication và @SpringBootConfiguration như thế nào?

`@SpringBootTest` là annotation chính để chạy các bài kiểm thử tích hợp (integration test) trong một ứng dụng Spring Boot. Nó cung cấp các tính năng của Spring Boot lên trên Spring Test Framework thông thường.

**`@SpringBootTest` làm những gì?**

* **Tự động tìm kiếm cấu hình**: Nó tự động tìm kiếm một lớp được chú thích bằng `@SpringBootConfiguration` để sử dụng làm nguồn cấu hình để tải `ApplicationContext`.
* **Tải Context**: Nó sử dụng `SpringBootContextLoader` để khởi động `ApplicationContext` thông qua `SpringApplication`, giống như khi chạy ứng dụng thật.
* **Cung cấp các môi trường web**: Cho phép bạn chạy test trong một môi trường web giả lập (`MOCK` - mặc định), hoặc với một server web thực sự chạy trên một cổng ngẫu nhiên (`RANDOM_PORT`) hoặc một cổng xác định (`DEFINED_PORT`).
* **Đăng ký các bean cho test**: Tự động đăng ký các bean hữu ích cho việc test như `TestRestTemplate` hoặc `WebTestClient` khi cần thiết.

**Nó tương tác với `@SpringBootApplication` và `@SpringBootConfiguration` như thế nào?**

Annotation `@SpringBootApplication` thực chất là một annotation tổng hợp, và một trong những annotation nó chứa là `@SpringBootConfiguration`.

`@SpringBootConfiguration` lại là một dạng thay thế cho `@Configuration` tiêu chuẩn, với ưu điểm là nó có thể được tự động tìm thấy bởi `@SpringBootTest`.

Khi bạn sử dụng `@SpringBootTest` mà không chỉ định rõ lớp cấu hình nào (`classes=...`), nó sẽ bắt đầu quét từ gói (package) của lớp test hiện tại và đi ngược lên các gói cha để tìm một lớp được chú thích bằng `@SpringBootConfiguration` (thường là lớp main của bạn với `@SpringBootApplication`).

**Ví dụ Code: Tương tác giữa các Annotation**

Cấu trúc thư mục:

```
com/
  example/
    MyApplication.java  // Chứa @SpringBootApplication
    service/
      MyService.java
    test/
      MyServiceIntegrationTest.java // Lớp test
```

Lớp ứng dụng chính:

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // Annotation này bao gồm @SpringBootConfiguration
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

Lớp Test:

```java
package com.example.test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertNotNull;

// @SpringBootTest sẽ tự động tìm thấy lớp MyApplication
// vì nó nằm trong gói cha (com.example)
@SpringBootTest
public class MyServiceIntegrationTest {

    @Autowired
    private MyService myService;

    @Test
    void contextLoads() {
        // Test này sẽ thành công nếu ApplicationContext được tải đúng cách
        // và MyService được tiêm vào.
        assertNotNull(myService);
    }
}
```