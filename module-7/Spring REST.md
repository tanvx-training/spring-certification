## Câu 1: REST là viết tắt của từ gì?

**REST** là viết tắt của **REpresentational State Transfer** (Truyền tải trạng thái đại diện).

Đây là một kiểu kiến trúc để thiết kế các ứng dụng phân tán. Trong kiến trúc REST, hệ thống yêu cầu (client) sẽ truy cập và thao tác trên các **biểu diễn văn bản (textual representations)** của tài nguyên web bằng cách sử dụng một tập hợp các **thao tác phi trạng thái (stateless operations)** được xác định trước.

Các tài nguyên web này được cung cấp thông qua URI (Uniform Resource Identifiers - Định danh tài nguyên đồng nhất) và thường được truy cập hoặc sửa đổi qua các phương thức HTTP. Hầu hết các dịch vụ REST sử dụng HTTP làm giao thức ứng dụng và JSON làm định dạng dữ liệu.

Việc tuân thủ các ràng buộc của REST mang lại các thuộc tính phi chức năng quan trọng cho hệ thống như hiệu suất (performance), khả năng mở rộng (scalability), tính đơn giản (simplicity), và độ tin cậy (reliability).

Các ràng buộc chính của REST bao gồm:

* Kiến trúc Client-Server
* Phi trạng thái (Statelessness)
* Khả năng lưu trữ đệm (Cacheability)
* Giao diện đồng nhất (Uniform interface)
* Hệ thống phân lớp (Layered system)

### Ví dụ Code

Đây là một ví dụ đơn giản về một REST Controller trong Spring Boot để trả về một lời chào. Nó sử dụng phương thức `GET` và một URI là `/api/greeting`.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class GreetingController {

    @GetMapping("/greeting")
    public String getGreeting() {
        return "Xin chào, đây là REST API đầu tiên của bạn!";
    }
}
```

-----

## Câu 2: Resource (Tài nguyên) là gì?

**Resource (Tài nguyên)** là một thông tin có tên, có thể truy cập được thông qua một URI. Nó có thể là bất cứ thứ gì: một tài liệu, hình ảnh, video, tệp văn bản, hoặc một đối tượng dữ liệu (ví dụ: một khách hàng, một sản phẩm).

REST sử dụng nhiều định dạng khác nhau để biểu diễn tài nguyên, và client có thể chỉ định định dạng mà mình mong muốn nhận được, ví dụ như **JSON, XML, Text, HTML**, thông qua HTTP header `Accept`.

Một tài nguyên có thể được cung cấp dưới dạng một đối tượng đơn lẻ (single resource) hoặc một tập hợp (collection of resources). Các tài nguyên cũng có thể có mối quan hệ với nhau, ví dụ như một khách hàng "chứa" nhiều địa chỉ.

### Ví dụ Code

Giả sử chúng ta có tài nguyên `Customer` (Khách hàng) và `Address` (Địa chỉ).

```java
// Lớp đại diện cho tài nguyên Customer
public class Customer {
    private Long id;
    private String name;
    // getters and setters
}

// Lớp đại diện cho tài nguyên Address
public class Address {
    private Long id;
    private String street;
    private String city;
    // getters and setters
}
```

Một `RestController` có thể cung cấp các endpoint để truy cập các tài nguyên này:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.Collections;
import java.util.List;

@RestController
public class CustomerController {

    // Lấy một tài nguyên đơn lẻ: khách hàng có ID là 1
    // URI: /customers/1
    @GetMapping("/customers/{id}")
    public Customer getCustomerById(@PathVariable Long id) {
        // Logic để tìm và trả về khách hàng
        return new Customer(id, "Nguyễn Văn A");
    }

    // Lấy một tập hợp tài nguyên: danh sách địa chỉ của khách hàng có ID là 1
    // URI: /customers/1/addresses
    @GetMapping("/customers/{id}/addresses")
    public List<Address> getAddressesForCustomer(@PathVariable Long id) {
        // Logic để tìm và trả về danh sách địa chỉ
        return Collections.singletonList(new Address(1L, "123 Đường Láng", "Hà Nội"));
    }
}
```

-----

## Câu 3: CRUD có nghĩa là gì?

**CRUD** là từ viết tắt của bốn hoạt động cơ bản trong việc quản lý dữ liệu:

* **C**reate (Tạo mới)
* **R**ead (Đọc/Truy xuất)
* **U**pdate (Cập nhật)
* **D**elete (Xóa)

Trong REST API sử dụng giao thức HTTP, các hoạt động CRUD này thường được ánh xạ tới các phương thức HTTP tiêu chuẩn như sau:

* **Create** → `POST` / `PUT`
* **Read** → `GET`
* **Update** → `PUT` / `PATCH`
* **Delete** → `DELETE`

### Ví dụ Code

Dưới đây là một ví dụ về một `ProductController` triển khai đầy đủ các hoạt động CRUD cho tài nguyên `Product`.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

// Lớp đại diện cho tài nguyên Product
class Product {
    private Long id;
    private String name;
    private double price;
    // Constructors, getters, setters
}

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final Map<Long, Product> products = new ConcurrentHashMap<>();
    private final AtomicLong counter = new AtomicLong();

    // READ (Đọc) - Lấy tất cả sản phẩm
    @GetMapping
    public List<Product> getAllProducts() {
        return new ArrayList<>(products.values());
    }

    // READ (Đọc) - Lấy sản phẩm theo ID
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        Product product = products.get(id);
        return (product != null) ? ResponseEntity.ok(product) : ResponseEntity.notFound().build();
    }

    // CREATE (Tạo mới) - Thêm một sản phẩm mới
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        long newId = counter.incrementAndGet();
        product.setId(newId);
        products.put(newId, product);
        return new ResponseEntity<>(product, HttpStatus.CREATED);
    }

    // UPDATE (Cập nhật) - Cập nhật toàn bộ thông tin sản phẩm
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(@PathVariable Long id, @RequestBody Product updatedProduct) {
        if (!products.containsKey(id)) {
            return ResponseEntity.notFound().build();
        }
        updatedProduct.setId(id);
        products.put(id, updatedProduct);
        return ResponseEntity.ok(updatedProduct);
    }

    // DELETE (Xóa) - Xóa một sản phẩm
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        if (products.remove(id) == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.noContent().build();
    }
}
```

-----

## Câu 4: REST có an toàn không? Bạn có thể làm gì để bảo mật nó?

Bản thân REST, với tư cách là một kiểu kiến trúc, **không tự nó cung cấp bất kỳ cơ chế bảo mật nào**. Do đó, mặc định REST không an toàn.

Tuy nhiên, vì REST khuyến khích cách tiếp cận phân lớp (layered approach), bảo mật có thể được thêm vào như một lớp riêng biệt trong hệ thống. Trong hệ sinh thái Spring, điều này có thể dễ dàng đạt được bằng cách sử dụng module **Spring Security**.

Để bảo mật một REST API, bạn có thể thực hiện các biện pháp sau:

1.  **Sử dụng HTTPS**: Mã hóa lưu lượng dữ liệu truyền đi giữa client và server để chống lại việc nghe lén.
2.  **Xác thực (Authentication)**: Xác định danh tính của người dùng. Các phương pháp phổ biến bao gồm:
    * Basic Authentication
    * OAuth 2.0
    * JSON Web Tokens (JWT)
3.  **Phân quyền (Authorization)**: Xác định những gì một người dùng đã được xác thực được phép làm. Ví dụ, sử dụng các vai trò (roles) như `ROLE_ADMIN`, `ROLE_USER` để giới hạn quyền truy cập vào các endpoint nhất định.

### Ví dụ Code

Đây là một ví dụ cơ bản về cấu hình Spring Security để yêu cầu xác thực cho tất cả các request đến API.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                // Yêu cầu tất cả các request phải được xác thực
                .anyRequest().authenticated()
            )
            // Sử dụng HTTP Basic Authentication
            .httpBasic(withDefaults());
        return http.build();
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        // Tạo một người dùng trong bộ nhớ để kiểm thử
        UserDetails user = User.withDefaultPasswordEncoder()
            .username("user")
            .password("password")
            .roles("USER")
            .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

-----

## Câu 5: REST có khả năng mở rộng và/hoặc tương tác không?

**Có**, REST có cả hai đặc tính này.

### Khả năng mở rộng (Scalability)

Khả năng mở rộng của dịch vụ RESTful đến từ các đặc điểm thiết kế sau:

* **Tính phi trạng thái (Statelessness)**: Mỗi request từ client phải chứa tất cả thông tin cần thiết để server xử lý nó mà không cần phải lưu trữ bất kỳ trạng thái nào của client (ví dụ: không dùng `HttpSession`). Điều này cho phép dễ dàng phân phối request đến bất kỳ node server nào, giúp việc mở rộng theo chiều ngang (horizontal scaling) trở nên đơn giản.
* **Hệ thống phân lớp (Layered Approach)**: Có thể thêm các thành phần trung gian (như Load Balancer, API Gateway, lớp bảo mật) vào hệ thống mà không cần thay đổi client.
* **Khả năng lưu trữ đệm (Cacheability)**: Các phản hồi có thể được lưu vào bộ đệm (cache) để tái sử dụng cho các request lặp lại, giúp giảm tải cho server và cải thiện thời gian phản hồi.

### Khả năng tương tác (Interoperability)

Dịch vụ REST có khả năng tương tác cao vì:

* **Dựa trên các tiêu chuẩn mở**: Việc truy cập tài nguyên qua URI và sử dụng các phương thức HTTP là tiêu chuẩn, không phụ thuộc vào bất kỳ công nghệ cụ thể nào. Client có thể được viết bằng bất kỳ ngôn ngữ nào (JavaScript, Python, Java, C++, v.v.).
* **Hỗ trợ nhiều định dạng dữ liệu**: Dữ liệu có thể được gửi và nhận ở nhiều định dạng (JSON, XML, v.v.). Client chỉ định định dạng mong muốn qua header `Accept`, và định dạng gửi đi qua header `Content-Type`.
* **Sử dụng các phương thức chuẩn**: Các hoạt động CRUD được xử lý bằng các phương thức HTTP chuẩn (GET, POST, PUT, DELETE), giúp mọi hệ thống đều có thể hiểu và làm việc cùng nhau.

### Ví dụ Code

Ví dụ sau minh họa khả năng tương tác bằng cách cho phép một endpoint sản xuất cả JSON và XML.

```java
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/data")
public class InteroperableController {

    // Endpoint này có thể trả về dữ liệu dạng JSON hoặc XML
    // tùy thuộc vào header 'Accept' của client
    @GetMapping(value = "/{id}", produces = { MediaType.APPLICATION_JSON_VALUE, MediaType.APPLICATION_XML_VALUE })
    public SomeData getData(@PathVariable Long id) {
        // Giả sử SomeData là một lớp POJO đã được cấu hình để tuần tự hóa
        return new SomeData(id, "Dữ liệu có khả năng tương tác");
    }
}

// Cần có thư viện Jackson XML (jackson-dataformat-xml) trong classpath để hỗ trợ XML
class SomeData {
    private Long id;
    private String content;
    // Constructors, getters, setters
}
```

-----

## Câu 6: REST sử dụng các phương thức HTTP nào?

REST sử dụng các phương thức HTTP tiêu chuẩn để thực hiện các hoạt động trên tài nguyên. Các phương thức chính bao gồm:

* **`GET`**: Dùng để **đọc** (Read) hoặc truy xuất một tài nguyên hoặc một danh sách tài nguyên. Đây là một hoạt động an toàn (safe) và không thay đổi trạng thái của tài nguyên.
* **`POST`**: Thường dùng để **tạo mới** (Create) một tài nguyên. Nó cũng có thể được sử dụng để kích hoạt một hành động trên server.
* **`PUT`**: Dùng để **tạo mới** (Create) hoặc **cập nhật toàn bộ** (Update) một tài nguyên đã tồn tại. Nếu tài nguyên tồn tại, nó sẽ bị thay thế hoàn toàn bởi dữ liệu mới. Nếu không, nó có thể được tạo mới.
* **`PATCH`**: Dùng để **cập nhật một phần** (Partial Update) của một tài nguyên đã tồn tại. Ví dụ, chỉ cập nhật địa chỉ email của một người dùng mà không cần gửi lại toàn bộ thông tin của họ.
* **`DELETE`**: Dùng để **xóa** (Delete) một tài nguyên đã tồn tại.

### Ví dụ Code

Ví dụ này mở rộng ví dụ CRUD ở trên, thêm phương thức `PATCH` để cập nhật một phần.

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.Map;

@RestController
@RequestMapping("/api/users")
public class UserController {
    // ... (Giả sử có một Map để lưu trữ người dùng)
    private Map<Long, User> users;

    // PATCH - Cập nhật một phần thông tin người dùng (ví dụ: chỉ email)
    @PatchMapping("/{id}")
    public ResponseEntity<User> partialUpdateUser(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        User user = users.get(id);
        if (user == null) {
            return ResponseEntity.notFound().build();
        }

        // Áp dụng các thay đổi từ request body
        if (updates.containsKey("email")) {
            user.setEmail((String) updates.get("email"));
        }
        if (updates.containsKey("fullName")) {
            user.setFullName((String) updates.get("fullName"));
        }
        
        users.put(id, user);
        return ResponseEntity.ok(user);
    }
}

class User {
    private Long id;
    private String email;
    private String fullName;
    // getters and setters
}
```

-----

## Câu 7: HttpMessageConverter là gì?

`HttpMessageConverter` là một interface cốt lõi trong Spring Framework, có nhiệm vụ **chuyển đổi dữ liệu** giữa đối tượng Java và nội dung của một HTTP request/response body.

Nó hoạt động "phía sau hậu trường" để thực hiện các công việc sau:

* **Đọc request**: Khi một request đến với một body (ví dụ trong phương thức `POST` hoặc `PUT`), `HttpMessageConverter` sẽ đọc nội dung (ví dụ: JSON, XML) và chuyển đổi nó thành một đối tượng Java (`@RequestBody`).
* **Ghi response**: Khi một phương thức trong controller trả về một đối tượng Java (`@ResponseBody` hoặc trong `@RestController`), `HttpMessageConverter` sẽ chuyển đổi đối tượng đó thành một định dạng cụ thể (ví dụ: JSON) và ghi nó vào HTTP response body.

Spring sẽ tự động chọn `HttpMessageConverter` phù hợp dựa trên:

1.  **`Content-Type` header** của request (để đọc).
2.  **`Accept` header** của request (để ghi).
3.  Loại đối tượng cần chuyển đổi.

Spring Boot tự động cấu hình sẵn nhiều converter phổ biến, như `MappingJackson2HttpMessageConverter` cho JSON và `MappingJackson2XmlHttpMessageConverter` cho XML.

### Ví dụ Code

Trong đoạn mã sau, bạn không cần phải tự mình phân tích cú pháp JSON. `HttpMessageConverter` sẽ tự động làm điều đó.

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class OrderController {

    // Khi nhận request POST đến /orders:
    // 1. Spring dùng HttpMessageConverter (ví dụ: Jackson) để đọc body của request.
    // 2. Nó chuyển đổi chuỗi JSON thành một đối tượng `Order`.
    // 3. Đối tượng `Order` được truyền vào phương thức này thông qua tham số `newOrder`.
    @PostMapping("/orders")
    public Receipt createOrder(@RequestBody Order newOrder) {
        // Logic xử lý đơn hàng...
        System.out.println("Đã nhận đơn hàng cho sản phẩm: " + newOrder.getProductName());

        // 4. Khi phương thức trả về đối tượng `Receipt`.
        // 5. HttpMessageConverter lại chuyển đối tượng `Receipt` thành chuỗi JSON.
        // 6. Chuỗi JSON này được gửi lại cho client trong body của response.
        return new Receipt("SUCCESS", "Đã xử lý đơn hàng " + newOrder.getProductName());
    }
}

// Các lớp POJO
class Order {
    private String productName;
    private int quantity;
    // getters and setters
}

class Receipt {
    private String status;
    private String message;
    // constructors, getters, setters
}
```

-----

## Câu 8: REST có thường là stateless (phi trạng thái) không?

**Có, statelessness (tính phi trạng thái) là một trong những ràng buộc cơ bản và quan trọng nhất của kiến trúc REST.**

Điều này có nghĩa là server **không lưu trữ bất kỳ thông tin ngữ cảnh nào về client** giữa các request. Mỗi request từ client phải chứa tất cả thông tin cần thiết để server có thể hiểu và xử lý nó một cách độc lập, mà không cần dựa vào bất kỳ request nào trước đó.

Nếu cần duy trì một trạng thái nào đó (ví dụ: thông tin phiên đăng nhập của người dùng), thì chính client phải chịu trách nhiệm gửi trạng thái đó trong mỗi request (ví dụ: gửi một token xác thực trong header `Authorization`). Server không được phép lưu trữ trạng thái này, ví dụ như trong `HttpSession`.

**Lợi ích chính của tính phi trạng thái là khả năng mở rộng (scalability)**, vì bất kỳ request nào cũng có thể được xử lý bởi bất kỳ server nào, giúp việc cân bằng tải trở nên hiệu quả.

### Ví dụ Code

#### Cách làm sai (Stateful - Có trạng thái)

Cách này vi phạm nguyên tắc của REST vì nó lưu trạng thái trong `HttpSession` trên server.

```java
import org.springframework.web.bind.annotation.GetMapping;
import jakarta.servlet.http.HttpSession;

@RestController
public class StatefulController {

    @GetMapping("/stateful/add")
    public String addItemToCart(HttpSession session) {
        Integer itemCount = (Integer) session.getAttribute("itemCount");
        if (itemCount == null) {
            itemCount = 0;
        }
        session.setAttribute("itemCount", itemCount + 1);
        return "Đã thêm sản phẩm. Tổng số lượng: " + (itemCount + 1);
    }
}
```

#### Cách làm đúng (Stateless - Phi trạng thái)

Client chịu trách nhiệm gửi thông tin xác thực (ở đây là JWT) trong mỗi request. Server xác thực request mà không cần lưu trữ session.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;

@RestController
public class StatelessController {

    @GetMapping("/stateless/profile")
    public String getUserProfile(@RequestHeader("Authorization") String token) {
        // Logic để giải mã token và lấy thông tin người dùng
        String userId = getUserIdFromToken(token);
        return "Thông tin hồ sơ của người dùng: " + userId;
    }

    private String getUserIdFromToken(String token) {
        // Giả lập việc giải mã token
        return "user123";
    }
}
```

-----

## Câu 9: @RequestMapping làm gì?

`@RequestMapping` là một annotation cực kỳ quan trọng và linh hoạt trong Spring MVC, dùng để **ánh xạ (map) các web request tới các phương thức xử lý (handler methods)** cụ thể trong controller.

Nó có thể được sử dụng ở cả cấp độ lớp (class-level) và cấp độ phương thức (method-level).

* **Ở cấp độ lớp**: Nó tạo ra một tiền tố URI chung cho tất cả các ánh xạ bên trong controller đó.
* **Ở cấp độ phương thức**: Nó xác định URI cụ thể và các điều kiện khác cho phương thức xử lý đó.

`@RequestMapping` cho phép bạn lọc request dựa trên các tiêu chí sau:

* `path` (hoặc `value`): URI của request (ví dụ: `/users`, `/products/{id}`).
* `method`: Phương thức HTTP (ví dụ: `GET`, `POST`, `PUT`).
* `params`: Các tham số trong request (ví dụ: `myParam=myValue`).
* `headers`: Các header trong request (ví dụ: `Content-Type=application/json`).
* `consumes`: Loại media type mà request có thể tiêu thụ (gửi đến server).
* `produces`: Loại media type mà response sẽ sản xuất (trả về cho client).

Spring cũng cung cấp các annotation rút gọn cho các phương thức HTTP phổ biến như `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, và `@PatchMapping`.

### Ví dụ Code

Ví dụ sau minh họa cách sử dụng `@RequestMapping` ở cả cấp độ lớp và phương thức, cũng như sử dụng annotation rút gọn `@GetMapping`.

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/customers") // Ánh xạ ở cấp độ lớp, tất cả các URI đều bắt đầu bằng /api/v1/customers
public class CustomerApiController {

    // Ánh xạ này sẽ xử lý request: GET /api/v1/customers
    @GetMapping
    public String getAllCustomers() {
        return "Trả về danh sách tất cả khách hàng";
    }

    // Ánh xạ này sẽ xử lý request: GET /api/v1/customers/123
    @GetMapping("/{customerId}")
    public String getCustomerById(@PathVariable String customerId) {
        return "Trả về thông tin khách hàng có ID: " + customerId;
    }
    
    // Sử dụng @RequestMapping với method cụ thể
    // Ánh xạ này sẽ xử lý request: POST /api/v1/customers
    @RequestMapping(method = RequestMethod.POST)
    public String createCustomer() {
        return "Tạo một khách hàng mới";
    }
}
```

-----

## Câu 10: @Controller và @RestController có phải là stereotype không? Stereotype là gì?

**Có, cả `@Controller` và `@RestController` đều là các annotation stereotype.**

**Stereotype annotation** là một annotation được dùng để đánh dấu một lớp với một "vai trò" (role) cụ thể trong ứng dụng. Khi Spring quét các thành phần (component scanning), nó sẽ phát hiện các lớp được đánh dấu bằng stereotype, khởi tạo chúng thành các Spring bean và đưa vào Application Context để quản lý.

Về cơ bản, stereotype cung cấp một ngữ nghĩa cho lớp, cho biết nó thực hiện chức năng gì. Các stereotype chính trong Spring bao gồm:

* `@Component`: Là stereotype gốc, chung nhất. Mọi stereotype khác đều kế thừa từ nó. Nó đánh dấu một lớp là một thành phần do Spring quản lý.
* `@Service`: Đánh dấu một lớp ở tầng dịch vụ (service layer), thường chứa logic nghiệp vụ chính của ứng dụng.
* `@Repository`: Đánh dấu một lớp ở tầng truy cập dữ liệu (data access layer), chịu trách nhiệm tương tác với cơ sở dữ liệu. Nó cũng giúp chuyển đổi các exception của persistence framework thành các exception nhất quán của Spring.
* `@Controller`: Đánh dấu một lớp là một controller trong mô hình MVC, thường trả về tên của một view để hiển thị.

### @RestController là gì?

`@RestController` là một stereotype đặc biệt, được giới thiệu trong Spring 4.0 để giúp việc xây dựng RESTful web services trở nên dễ dàng hơn.

Nó là sự kết hợp của hai annotation:

1.  **`@Controller`**: Đánh dấu lớp là một web controller.
2.  **`@ResponseBody`**: Tự động áp dụng cho tất cả các phương thức xử lý request trong controller đó. `@ResponseBody` báo cho Spring biết rằng giá trị trả về của phương thức nên được ghi trực tiếp vào response body (thường được chuyển đổi thành JSON hoặc XML bởi `HttpMessageConverter`), thay vì được giải quyết thành một tên view.

Nói cách khác, `@RestController` = `@Controller` + `@ResponseBody`.

### Ví dụ Code

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

// --- CÁCH LÀM TRUYỀN THỐNG VỚI @Controller VÀ @ResponseBody ---
@Controller
public class TraditionalRestController {

    @GetMapping("/traditional-hello")
    @ResponseBody // Phải thêm @ResponseBody để trả về dữ liệu thay vì tên view
    public String sayHello() {
        return "Hello from a traditional controller!";
    }
}


// --- CÁCH LÀM HIỆN ĐẠI VÀ TIỆN LỢI VỚI @RestController ---
@RestController // Bao gồm cả @Controller và @ResponseBody
public class ModernRestController {

    @GetMapping("/modern-hello")
    // Không cần @ResponseBody ở đây nữa
    public String sayHello() {
        return "Hello from a @RestController!";
    }
}
```


## Câu 11: Sự khác biệt giữa @Controller và @RestController là gì?

[cite\_start]Sự khác biệt chính nằm ở giá trị trả về của các phương thức xử lý (handler methods). [cite: 226, 227]

* [cite\_start]**`@Controller`**: Đây là một annotation của Spring MVC truyền thống. [cite: 226] [cite\_start]Nó đánh dấu một lớp là một controller, và các phương thức trong đó thường trả về một đối tượng `Model` và tên của một `View` (ví dụ: một trang HTML). [cite: 226] [cite\_start]Sau đó, một template engine (như Thymeleaf) sẽ sử dụng chúng để render giao diện người dùng. [cite: 226]

* [cite\_start]**`@RestController`**: Đây là một annotation chuyên dụng để xây dựng các RESTful API. [cite: 227] [cite\_start]Nó là sự kết hợp của `@Controller` và `@ResponseBody`. [cite: 228] [cite\_start]Khi sử dụng `@RestController`, giá trị trả về của các phương thức sẽ tự động được tuần tự hóa (serialized) thành một định dạng như JSON hoặc XML và được ghi trực tiếp vào phần thân (body) của HTTP response. [cite: 227, 229]

[cite\_start]Về cơ bản: `@RestController` = `@Controller` + `@ResponseBody`. [cite: 228]

### Ví dụ Code

**Sử dụng `@Controller` (MVC truyền thống)**

Controller này trả về tên của một view là `user-view`. Spring sẽ tìm một tệp template (ví dụ: `user-view.html`) để render.

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class UserMvcController {

    @GetMapping("/user-info")
    public String getUserInfo(@RequestParam("name") String name, Model model) {
        // Thêm dữ liệu vào model để view có thể sử dụng
        model.addAttribute("username", name);
        model.addAttribute("email", name.toLowerCase() + "@example.com");
        
        // Trả về tên của view
        return "user-view"; 
    }
}
```

**Sử dụng `@RestController` (REST API)**

Controller này trả về một đối tượng `User`. Spring sẽ tự động chuyển đổi đối tượng này thành JSON và gửi lại cho client.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

// Lớp POJO đại diện cho dữ liệu
class User {
    public String username;
    public String email;
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
}

@RestController
public class UserApiController {

    @GetMapping("/api/users/{name}")
    public User getUserData(@PathVariable String name) {
        // Trả về một đối tượng, đối tượng này sẽ được chuyển thành JSON
        return new User(name, name.toLowerCase() + "@example.com");
    }
}
```

-----

## Câu 12: Khi nào bạn cần dùng @ResponseBody?

[cite\_start]Bạn cần sử dụng annotation **`@ResponseBody`** khi bạn muốn giá trị trả về của một phương thức trong controller được ghi trực tiếp vào phần thân của HTTP response. [cite: 100] [cite\_start]Thay vì diễn giải giá trị trả về như là tên của một view, Spring sẽ sử dụng một `HttpMessageConverter` để tuần tự hóa (serialize) giá trị đó (thường là thành JSON hoặc XML). [cite: 100, 101]

[cite\_start]`@ResponseBody` rất hữu ích khi xây dựng các backend API, chẳng hạn như REST API. [cite: 106]

[cite\_start]Tuy nhiên, bạn **không cần** sử dụng `@ResponseBody` nếu đã sử dụng `@RestController` cho cả lớp, vì `@RestController` đã bao gồm `@ResponseBody` trong định nghĩa của nó. [cite: 107, 108]

Nó có thể được sử dụng ở các vị trí sau:

* [cite\_start]Trên một phương thức. [cite: 104]
* [cite\_start]Trên một lớp (sẽ áp dụng cho tất cả các phương thức trong lớp đó). [cite: 103]
* [cite\_start]Trên một annotation khác (như cách `@RestController` được tạo ra). [cite: 105]

### Ví dụ Code

Ví dụ dưới đây cho thấy cách sử dụng `@ResponseBody` trên một phương thức trong một `@Controller` thông thường.

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller // Đây là một Controller thông thường, không phải RestController
public class ProductController {

    // Lớp POJO để làm dữ liệu trả về
    public record Product(long id, String name) {}

    // Phương thức này trả về một trang HTML (tên view)
    @GetMapping("/products/page")
    public String getProductPage() {
        return "product-list"; // Giả sử có một template tên là product-list.html
    }

    // Phương thức này, nhờ có @ResponseBody, sẽ trả về dữ liệu JSON
    @GetMapping("/api/products/featured")
    @ResponseBody // Bắt buộc phải có để trả về dữ liệu thay vì tên view
    public Product getFeaturedProduct() {
        return new Product(101L, "Laptop Siêu Cấp");
    }
}
```

-----

## Câu 13: Các mã trạng thái HTTP trả về cho một hoạt động GET, POST, PUT hoặc DELETE thành công là gì?

Dưới đây là các mã trạng thái (status codes) HTTP thường được sử dụng cho các hoạt động thành công trong một REST API:

* **GET**:

    * [cite\_start]**200 (OK)**: Được trả về khi tài nguyên được tìm thấy và nội dung của nó được gửi lại thành công. [cite: 470]

* **POST**:

    * [cite\_start]**201 (Created)**: Được trả về khi một tài nguyên mới đã được tạo thành công trên server. [cite: 472]
    * [cite\_start]**200 (OK)**: Khi một quá trình xử lý được thực thi thành công nhưng không có tài nguyên mới nào được tạo. [cite: 473]
    * [cite\_start]**204 (No Content)**: Khi một quá trình xử lý được thực thi thành công và không có nội dung nào được trả về trong phần thân của response. [cite: 474]

* **PUT**:

    * [cite\_start]**201 (Created)**: Khi một tài nguyên mới được tạo bằng phương thức PUT. [cite: 475]
    * [cite\_start]**200 (OK)**: Khi một tài nguyên đã tồn tại được cập nhật thành công và nội dung đã cập nhật được trả về. [cite: 476]
    * [cite\_start]**204 (No Content)**: Khi một tài nguyên đã tồn tại được cập nhật thành công và không có nội dung nào được trả về. [cite: 477]

* **DELETE**:

    * [cite\_start]**204 (No Content)**: Được trả về khi tài nguyên đã được xóa thành công. [cite: 479]

[cite\_start]Đối với các hoạt động bất đồng bộ (asynchronous), mã **202 (Accepted)** có thể được sử dụng để cho biết yêu cầu đã được chấp nhận để xử lý. [cite: 480]

### Ví dụ Code

Ví dụ này sử dụng `ResponseEntity` để kiểm soát mã trạng thái HTTP trả về một cách tường minh.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/books")
public class BookController {

    // POST: Tạo một sách mới
    @PostMapping
    public ResponseEntity<String> createBook(@RequestBody String book) {
        // Logic để lưu sách...
        // Trả về 201 Created
        return new ResponseEntity<>("Sách đã được tạo", HttpStatus.CREATED);
    }

    // PUT: Cập nhật một sách đã có
    @PutMapping("/{id}")
    public ResponseEntity<String> updateBook(@PathVariable int id, @RequestBody String book) {
        // Logic để cập nhật sách...
        // Trả về 200 OK
        return ResponseEntity.ok("Sách đã được cập nhật");
    }

    // DELETE: Xóa một sách
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable int id) {
        // Logic để xóa sách...
        // Trả về 204 No Content
        return ResponseEntity.noContent().build();
    }
}
```

-----

## Câu 14: Khi nào bạn cần dùng @ResponseStatus?

[cite\_start]Bạn cần sử dụng annotation **`@ResponseStatus`** khi muốn **ghi đè (override)** mã trạng thái HTTP mặc định được trả về từ một phương thức xử lý trong controller. [cite: 237]

Thông thường, một phương thức thành công sẽ trả về `200 OK`. `@ResponseStatus` cho phép bạn thay đổi điều này thành một mã trạng thái khác (ví dụ: `201 Created`) mà không cần sử dụng `ResponseEntity`.

`@ResponseStatus` có thể được áp dụng tại:

* [cite\_start]Một phương thức trong controller. [cite: 239]
* [cite\_start]Toàn bộ lớp controller. [cite: 238]
* [cite\_start]Một lớp Exception (ngoại lệ). [cite: 240]

Nó cho phép bạn thiết lập:

* [cite\_start]Mã trạng thái HTTP (ví dụ: `HttpStatus.CREATED`). [cite: 242]
* [cite\_start]Một thông báo lý do (reason message) sẽ được sử dụng trong response nếu có lỗi. [cite: 243]

### Ví dụ Code

**1. Sử dụng trên một phương thức**

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/gadgets")
public class GadgetController {

    // Thay vì trả về 200 OK mặc định, phương thức này sẽ trả về 201 Created
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public String createGadget(@RequestBody String gadget) {
        // Logic tạo mới...
        return "Tạo mới tiện ích thành công!";
    }
}
```

**2. Sử dụng trên một lớp Exception**

Đây là một cách rất hay để xử lý lỗi một cách tập trung. Khi exception này được ném ra từ bất kỳ controller nào, Spring sẽ tự động trả về `404 Not Found`.

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

// Định nghĩa một exception tùy chỉnh
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Tài nguyên không tồn tại trong hệ thống")
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

@RestController
public class ItemController {
    @GetMapping("/items/{id}")
    public String findItem(@PathVariable int id) {
        if (id > 100) {
            // Khi ném exception này, client sẽ nhận được lỗi 404
            throw new ResourceNotFoundException("Không tìm thấy item với id: " + id);
        }
        return "Thông tin item " + id;
    }
}
```

-----

## Câu 15: Bạn cần @ResponseBody ở đâu? Còn @RequestBody thì sao?

Cả hai annotation này đều đóng vai trò quan trọng trong việc xử lý dữ liệu cho các REST API.

* **`@RequestBody`**:

    * [cite\_start]**Mục đích**: Dùng để **liên kết (bind)** phần thân của một HTTP request đến một tham số của phương thức trong controller. [cite: 9] [cite\_start]`HttpMessageConverter` sẽ được sử dụng để chuyển đổi nội dung của request (ví dụ: JSON) thành một đối tượng Java. [cite: 10]
    * [cite\_start]**Vị trí**: Chỉ có thể được sử dụng trên một **tham số của phương thức** trong controller. [cite: 17]
    * [cite\_start]**Tính năng bổ sung**: Có thể kết hợp với `@Valid` để kích hoạt Bean Validation tự động. [cite: 10]

* **`@ResponseBody`**:

    * [cite\_start]**Mục đích**: Dùng để báo cho Spring biết rằng giá trị trả về của một phương thức nên được **tuần tự hóa (serialize)** và ghi trực tiếp vào phần thân của HTTP response. [cite: 6] [cite\_start]Nó cũng sử dụng `HttpMessageConverter` để thực hiện việc chuyển đổi. [cite: 8]
    * [cite\_start]**Vị trí**: Có thể được sử dụng trên một **phương thức**, trên một **lớp**, hoặc trên một **annotation khác**. [cite: 12, 13, 14, 15]
    * [cite\_start]**Lưu ý**: Nếu bạn sử dụng `@RestController`, bạn không cần phải thêm `@ResponseBody` nữa. [cite: 8]

### Ví dụ Code

Ví dụ này minh họa cả hai annotation trong cùng một phương thức xử lý.

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

// Lớp POJO cho request
class CalculationRequest {
    public int numberA;
    public int numberB;
}

// Lớp POJO cho response
class CalculationResponse {
    public int sum;
    public CalculationResponse(int sum) { this.sum = sum; }
}

@RestController // Đã bao gồm @ResponseBody cho tất cả các phương thức
public class CalculatorController {

    @PostMapping("/api/calculate/sum")
    public CalculationResponse calculateSum(
        // @RequestBody: Chuyển đổi JSON từ request body thành đối tượng CalculationRequest
        @RequestBody CalculationRequest request 
    ) {
        int result = request.numberA + request.numberB;

        // Giá trị trả về (CalculationResponse) sẽ được chuyển đổi thành JSON và
        // ghi vào response body nhờ có @RestController
        return new CalculationResponse(result);
    }
}
```

Khi bạn gửi một request POST đến `/api/calculate/sum` với body là `{"numberA": 5, "numberB": 10}`, bạn sẽ nhận lại response với body là `{"sum": 15}`.

-----

## Câu 16: Nếu bạn thấy một đoạn mã Controller ví dụ, bạn có hiểu nó đang làm gì không?

Có, việc hiểu một đoạn mã controller trong Spring trở nên khá đơn giản nếu bạn nắm vững ý nghĩa của các annotation chính.

Một controller điển hình cho REST API thường bao gồm các thành phần sau:

1.  **Định nghĩa Controller**:

    * [cite\_start]`@RestController`: Đánh dấu lớp là một REST controller. [cite: 309] [cite\_start]Điều này ngụ ý rằng mỗi phương thức sẽ tự động có hành vi của `@ResponseBody`. [cite: 309, 326, 352]
    * [cite\_start]`@RequestMapping("/api")`: Đặt một đường dẫn gốc (prefix) cho tất cả các endpoint trong controller này. [cite: 353]

2.  **Phương thức xử lý (Handler Method)**:

    * [cite\_start]`@GetMapping`, `@PostMapping`, v.v.: Các annotation này ánh xạ các request HTTP đến các phương thức cụ thể dựa trên phương thức (GET, POST) và đường dẫn. [cite: 310, 311, 312, 313]
    * [cite\_start]`@PathVariable`: Dùng để trích xuất một giá trị từ URI. [cite: 334] Ví dụ: `/users/{id}`.
    * [cite\_start]`@RequestParam`: Dùng để lấy một tham số từ chuỗi truy vấn (query string). [cite: 333] Ví dụ: `/users?role=admin`.
    * [cite\_start]`@RequestBody`: Dùng để lấy toàn bộ phần thân (body) của request và chuyển đổi nó thành một đối tượng Java. [cite: 322]
    * [cite\_start]`@Valid`: Thường đi kèm với `@RequestBody` để kích hoạt việc xác thực (validation) cho đối tượng đó. [cite: 323, 362]

3.  **Xử lý Response**:

    * **ResponseEntity**: Một đối tượng bao bọc cho phép bạn kiểm soát toàn bộ HTTP response, bao gồm mã trạng thái, headers, và body.
    * [cite\_start]`@ResponseStatus`: Một cách khác để tùy chỉnh mã trạng thái HTTP trả về. [cite: 327]

4.  **Xử lý lỗi**:

    * [cite\_start]`@ExceptionHandler`: Định nghĩa một phương thức để xử lý các exception cụ thể được ném ra từ trong controller đó. [cite: 343]
    * [cite\_start]`@ControllerAdvice`: Cho phép bạn định nghĩa các trình xử lý lỗi toàn cục (`@ExceptionHandler`), áp dụng cho tất cả các controller trong ứng dụng. [cite: 344]

### Ví dụ Code Phân Tích

Hãy phân tích một ví dụ controller hoàn chỉnh:

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import javax.validation.Valid;

// 1. @RestController: Định nghĩa đây là một REST controller.
//    Mọi giá trị trả về sẽ được tuần tự hóa vào response body.
@RestController
// 2. @RequestMapping: Tất cả các endpoint sẽ có tiền tố /api/products
@RequestMapping("/api/products") 
public class ProductApiController {

    // 3. @GetMapping: Xử lý request GET đến /api/products/{id}
    //    @PathVariable trích xuất 'id' từ đường dẫn.
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        // Logic tìm sản phẩm...
        return new Product(id, "Sản phẩm " + id);
    }
    
    // 4. @PostMapping: Xử lý request POST đến /api/products
    @PostMapping
    public ResponseEntity<Product> createProduct(
        // 5. @RequestBody: Chuyển đổi body của request thành đối tượng Product.
        // 6. @Valid: Yêu cầu Spring xác thực đối tượng product này.
        @RequestBody @Valid Product product
    ) {
        // Logic lưu sản phẩm...
        Product savedProduct = save(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedProduct);
    }
}
```

-----

## Câu 17: Bạn có cần Spring MVC trong classpath không?

[cite\_start]**Có, bạn cần có Spring MVC trong classpath để một REST API hoạt động chính xác.** [cite: 251]

[cite\_start]Mặc dù các annotation chính bạn sử dụng để xây dựng REST API (như `@RestController`, `@RequestMapping`, `@GetMapping`, `@RequestBody`) nằm trong module **`spring-web`**, nhưng module này không phụ thuộc vào `spring-webmvc`. [cite: 257]

[cite\_start]Tuy nhiên, để một request có thể được định tuyến và xử lý bởi một `@RestController`, cần phải có **`DispatcherServlet`** được khởi tạo. [cite: 258] `DispatcherServlet` chính là "trái tim" của Spring MVC, chịu trách nhiệm nhận tất cả các request đến và điều phối chúng đến các controller phù hợp. [cite\_start]`DispatcherServlet` nằm trong module **`spring-webmvc`**. [cite: 258]

[cite\_start]Do đó, `spring-webmvc` là một phụ thuộc bắt buộc lúc chạy (runtime), mặc dù không cần thiết lúc biên dịch (compile time). [cite: 252, 259]

Khi bạn sử dụng Spring Boot, việc này trở nên vô cùng đơn giản.

### Ví dụ Code (Maven)

Khi bạn thêm starter `spring-boot-starter-web` vào tệp `pom.xml`, nó sẽ tự động kéo cả `spring-web` và `spring-webmvc` vào dưới dạng các phụ thuộc bắc cầu (transitive dependencies), vì vậy bạn không cần phải khai báo chúng một cách tường minh.

```xml
<dependencies>
    
    [cite_start][cite: 191, 197]
    <dependency>
        [cite_start]<groupId>org.springframework.boot</groupId> [cite: 193]
        [cite_start]<artifactId>spring-boot-starter-web</artifactId> [cite: 194]
    </dependency>
    
</dependencies>
```

`spring-boot-starter-web` sẽ bao gồm:

* [cite\_start]`spring-web` (chứa các annotation REST cơ bản). [cite: 200]
* [cite\_start]`spring-webmvc` (chứa `DispatcherServlet` và toàn bộ framework MVC). [cite: 201]
* [cite\_start]`spring-boot-starter-json` (hỗ trợ Jackson để xử lý JSON). [cite: 202]
* [cite\_start]`spring-boot-starter-tomcat` (cung cấp một server Tomcat nhúng). [cite: 204]

-----

## Câu 18: Bạn sẽ sử dụng Spring Boot starter nào cho một ứng dụng Spring REST?

[cite\_start]Để tạo một ứng dụng Spring REST, bạn nên sử dụng **`spring-boot-starter-web`**. [cite: 191]

[cite\_start]Đây là starter tiêu chuẩn cho việc xây dựng các ứng dụng web, bao gồm cả các ứng dụng RESTful, sử dụng Spring MVC. [cite: 197] [cite\_start]Theo mặc định, nó sử dụng Tomcat làm server nhúng. [cite: 198]

Việc thêm starter này vào dự án của bạn sẽ tự động cấu hình tất cả các phụ thuộc cần thiết để bạn có thể bắt đầu xây dựng các endpoint ngay lập tức.

### Ví dụ Code (Maven)

Bạn chỉ cần thêm phụ thuộc sau vào tệp `pom.xml` của mình:

```xml
<dependency>
    [cite_start]<groupId>org.springframework.boot</groupId> [cite: 193]
    [cite_start]<artifactId>spring-boot-starter-web</artifactId> [cite: 194]
</dependency>
```

Starter này sẽ tự động thêm các phụ thuộc quan trọng sau vào classpath của bạn:

* [cite\_start]**`spring-web`**: Cung cấp các tích hợp web cơ bản. [cite: 200]
* [cite\_start]**`spring-webmvc`**: Cung cấp framework Spring MVC cho việc xử lý request, bao gồm cả các REST API. [cite: 201]
* [cite\_start]**`spring-boot-starter-json`**: Cung cấp thư viện Jackson để xử lý việc chuyển đổi đối tượng Java sang JSON và ngược lại. [cite: 202, 203]
* [cite\_start]**`spring-boot-starter-tomcat`**: Cung cấp một server Tomcat nhúng để bạn có thể chạy ứng dụng của mình mà không cần triển khai lên một server riêng biệt. [cite: 204, 205]

-----

## Câu 19: Ưu điểm của RestTemplate là gì?

[cite\_start]`RestTemplate` là một client HTTP **đồng bộ (synchronous)** của Spring, được dùng để thực hiện các HTTP request từ một ứng dụng Spring đến các dịch vụ khác. [cite: 141]

Các ưu điểm chính của `RestTemplate` bao gồm:

* [cite\_start]**Đơn giản (Simplicity)**: Nó cung cấp một API cấp cao, dễ sử dụng, che giấu sự phức tạp của các thư viện HTTP client bên dưới (như `HttpURLConnection` của JDK hay Apache `HttpComponents`). [cite: 142, 161, 163]
* [cite\_start]**Tự động chuyển đổi đối tượng**: Nó có thể tự động tuần tự hóa (serialize) các đối tượng Java thành JSON/XML để gửi trong request body và giải tuần tự hóa (deserialize) các response body thành các đối tượng Java. [cite: 146, 162] [cite\_start]Nó tự động đăng ký các `HttpMessageConverters` cần thiết (như Jackson, JAXB2). [cite: 147, 148]
* [cite\_start]**Hỗ trợ các phương thức HTTP phổ biến**: Cung cấp các phương thức tiện lợi cho các hoạt động GET, POST, PUT, DELETE, PATCH, v.v. [cite: 164]
* [cite\_start]**Hỗ trợ URI Templates**: Dễ dàng truyền tham số vào URI. [cite: 149, 170]
* [cite\_start]**Linh hoạt và có thể mở rộng**: Cho phép tùy chỉnh việc xử lý lỗi bằng cách cung cấp một `ResponseErrorHandler` tùy chỉnh và cho phép đăng ký các `HttpMessageConverters` bổ sung. [cite: 167, 168]

**Lưu ý quan trọng**: Kể từ Spring 5, `RestTemplate` đã được đưa vào chế độ bảo trì (maintenance mode). `WebClient`, một client HTTP không đồng bộ (non-blocking) và phản ứng (reactive), là sự lựa chọn được khuyến nghị cho các ứng dụng mới.

### Ví dụ Code

Dưới đây là cách sử dụng `RestTemplate` để gọi một API bên ngoài.

```java
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

// Lớp POJO để hứng dữ liệu từ API
// Đảm bảo các thuộc tính khớp với JSON trả về
public class Post {
    private int userId;
    private int id;
    private String title;
    private String body;
    // Getters and setters...
}

@Service
public class ExternalApiService {

    private final RestTemplate restTemplate;

    public ExternalApiService() {
        this.restTemplate = new RestTemplate();
    }

    // Lấy một bài viết cụ thể từ API JSONPlaceholder
    public Post getPostById(int postId) {
        // Định nghĩa URL của API với một placeholder {id}
        String apiUrl = "https://jsonplaceholder.typicode.com/posts/{id}";

        // Gọi API, RestTemplate sẽ tự động thay thế {id} bằng giá trị của postId
        // và chuyển đổi JSON response thành đối tượng Post.
        Post post = restTemplate.getForObject(apiUrl, Post.class, postId);
        
        return post;
    }
}
```

-----

## Câu 20: Nếu bạn thấy một ví dụ sử dụng RestTemplate, bạn có hiểu nó đang làm gì không?

Có, `RestTemplate` có một API khá trực quan và dễ hiểu, được tổ chức xoay quanh các phương thức HTTP.

Khi xem một ví dụ sử dụng `RestTemplate`, bạn cần chú ý đến tên của phương thức được gọi, vì nó thường cho biết:

1.  **Phương thức HTTP** nào đang được sử dụng (GET, POST, v.v.).
2.  **Loại dữ liệu** nào được mong đợi trả về.

`RestTemplate` có thể trả về kết quả theo hai cách chính:

* [cite\_start]**Dưới dạng một đối tượng (Object)**: Các phương thức có tên chứa `forObject` (ví dụ: `getForObject`, `postForObject`) sẽ trực tiếp trả về đối tượng đã được giải tuần tự hóa từ response body. [cite: 57, 76, 82]
* [cite\_start]**Dưới dạng một `ResponseEntity`**: Các phương thức có tên chứa `forEntity` (ví dụ: `getForEntity`, `postForEntity`) sẽ trả về một đối tượng `ResponseEntity`. [cite: 58, 77, 83] [cite\_start]Đối tượng này bao bọc cả response body, mã trạng thái HTTP, và các HTTP headers. [cite: 59, 60, 61]

Dưới đây là một số phương thức phổ biến được phân loại theo HTTP verb:

* [cite\_start]**GET**: `getForObject()`, `getForEntity()` [cite: 76, 77]
* [cite\_start]**POST**: `postForObject()`, `postForEntity()`, `postForLocation()` [cite: 81, 82, 83]
* [cite\_start]**PUT**: `put()` [cite: 85]
* [cite\_start]**DELETE**: `delete()` [cite: 89]
* [cite\_start]**Linh hoạt nhất**: `exchange()` cho phép bạn chỉ định mọi thứ một cách tường minh (phương thức HTTP, headers, body) và trả về một `ResponseEntity`. [cite: 71]

### Ví dụ Code

Ví dụ sau minh họa sự khác biệt giữa `getForObject` và `getForEntity`.

```java
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class ApiClient {

    private final RestTemplate restTemplate = new RestTemplate();
    private final String apiUrl = "https://jsonplaceholder.typicode.com/todos/1";

    // Sử dụng getForObject: Chỉ quan tâm đến nội dung (body)
    public void fetchDataAsObject() {
        System.out.println("--- Gọi API với getForObject ---");
        // Trực tiếp nhận đối tượng Todo, không biết gì về status code hay headers
        Todo todo = restTemplate.getForObject(apiUrl, Todo.class);
        System.out.println("Tiêu đề công việc: " + todo.getTitle());
    }

    // Sử dụng getForEntity: Quan tâm đến cả body, status code và headers
    public void fetchDataAsEntity() {
        System.out.println("\n--- Gọi API với getForEntity ---");
        // Nhận một đối tượng ResponseEntity
        ResponseEntity<Todo> response = restTemplate.getForEntity(apiUrl, Todo.class);
        
        System.out.println("Status Code: " + response.getStatusCode());
        System.out.println("Content-Type Header: " + response.getHeaders().getContentType());
        
        // Lấy body từ đối tượng response
        Todo todo = response.getBody();
        if (todo != null) {
            System.out.println("Tiêu đề công việc: " + todo.getTitle());
        }
    }
}

// Lớp POJO để hứng dữ liệu
class Todo {
    private int id;
    private String title;
    private boolean completed;
    // Getters and setters...
}
```