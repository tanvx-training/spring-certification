## 1\. MVC là viết tắt của một mẫu thiết kế. Nó là gì và ý tưởng đằng sau nó là gì?

**MVC** là viết tắt của **Model-View-Controller**. Đây là một mẫu kiến trúc phần mềm giúp tách biệt ứng dụng thành ba thành phần chính có liên kết với nhau, giúp cho việc phát triển và bảo trì dễ dàng hơn.

* **Model**: Là trái tim của ứng dụng, chịu trách nhiệm quản lý dữ liệu và logic nghiệp vụ. Nó không biết gì về cách dữ liệu sẽ được hiển thị. Các công việc chính của Model bao gồm:

    * Truy cập dữ liệu (Data Access).
    * Định nghĩa cấu trúc dữ liệu (Data Structures).
    * Xử lý logic nghiệp vụ (Business Logic).
    * Thực hiện các thao tác CRUD (Create, Read, Update, Delete).

* **View**: Chịu trách nhiệm hiển thị dữ liệu cho người dùng. Nó nhận dữ liệu từ Model và trình bày chúng. Cùng một dữ liệu có thể được hiển thị theo nhiều cách khác nhau. Trong Spring, View có thể là các công nghệ như Thymeleaf, JSP, FreeMarker, v.v.

* **Controller**: Đóng vai trò trung gian giữa Model và View. Nó tiếp nhận yêu cầu từ người dùng, tương tác với Model để xử lý yêu cầu đó, và sau đó quyết định View nào sẽ được dùng để trả về kết quả cho người dùng.

**Ý tưởng chính** của MVC là **phân tách các mối quan tâm (Separation of Concerns)**. Điều này giúp giảm sự phụ thuộc lẫn nhau giữa các thành phần, tăng khả năng tái sử dụng code, dễ bảo trì và mở rộng.

### Ví dụ Code

Đây là một ví dụ đơn giản về một `UserController` trong Spring MVC.

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class UserController {

    // Phương thức này xử lý yêu cầu GET đến "/user"
    @GetMapping("/user")
    public String getUserProfile(@RequestParam("id") String userId, Model model) {
        // 1. Controller nhận yêu cầu và tham số 'id'

        // 2. Tương tác với Model (ở đây ta giả lập việc lấy dữ liệu)
        // Trong thực tế, bạn sẽ gọi một service để truy vấn database
        User user = new User("John Doe", "john.doe@example.com");

        // 3. Thêm dữ liệu vào đối tượng Model để truyền cho View
        model.addAttribute("userName", user.getName());
        model.addAttribute("userEmail", user.getEmail());

        // 4. Trả về tên của View (ví dụ: userProfile.html) để hiển thị
        return "userProfile";
    }

    // Một lớp giả lập cho User
    private static class User {
        private String name;
        private String email;

        public User(String name, String email) {
            this.name = name;
            this.email = email;
        }

        public String getName() { return name; }
        public String getEmail() { return email; }
    }
}
```

Trong ví dụ trên:

* `UserController` là **Controller**.
* Đối tượng `Model` (do Spring cung cấp) và dữ liệu `User` là **Model**.
* Chuỗi `"userProfile"` là tên của **View** (ví dụ: một file template Thymeleaf) sẽ được render để trả về cho người dùng.

-----

## 2\. DispatcherServlet là gì và nó được dùng để làm gì?

**DispatcherServlet** là thành phần cốt lõi của Spring MVC. Nó hoạt động như một **Front Controller**, nghĩa là nó là servlet duy nhất xử lý tất cả các yêu cầu HTTP đến ứng dụng của bạn.

**Công dụng chính của DispatcherServlet là:**

1.  **Tiếp nhận yêu cầu**: Nó chặn tất cả các yêu cầu HTTP đến.
2.  **Ủy quyền xử lý**: Dựa trên URL của yêu cầu, nó tìm ra `Controller` và phương thức phù hợp để xử lý yêu cầu đó. Quá trình này được thực hiện với sự trợ giúp của các `HandlerMapping`.
3.  **Thực thi Controller**: Sau khi tìm thấy handler (phương thức controller), `DispatcherServlet` sử dụng một `HandlerAdapter` để thực thi phương thức đó.
4.  **Xử lý kết quả**: Nó nhận kết quả trả về từ Controller (thường là tên của một View và dữ liệu Model).
5.  **Render View**: Nó sử dụng các `ViewResolver` để tìm và render View tương ứng, sau đó trả về phản hồi (response) hoàn chỉnh cho người dùng.

Về cơ bản, `DispatcherServlet` điều phối toàn bộ luồng xử lý một yêu cầu trong Spring MVC.

### Ví dụ Code

Để `DispatcherServlet` hoạt động, nó cần được đăng ký và cấu hình. Trong một ứng dụng Spring Boot hiện đại, việc này được tự động hóa. Tuy nhiên, nếu cấu hình thủ công (ví dụ trong file `web.xml`), nó sẽ trông như sau:

```xml
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/dispatcher-config.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

Trong cấu hình dựa trên Java, nó sẽ trông như thế này:

```java
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null; // Cấu hình cho tầng business, service
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        // Cấu hình cho tầng web (Controllers, ViewResolvers)
        return new Class<?>[] { WebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        // Ánh xạ DispatcherServlet để xử lý tất cả các yêu cầu
        return new String[] { "/" };
    }
}
```

-----

## 3\. Web application context là gì? Nó cung cấp những scope (phạm vi) bổ sung nào?

**Web Application Context** là một `ApplicationContext` (bộ chứa IoC của Spring) được mở rộng dành riêng cho các ứng dụng web. Nó kế thừa tất cả các tính năng của một `ApplicationContext` thông thường và thêm vào các tính năng liên quan đến web, đặc biệt là quyền truy cập vào `ServletContext`.

Ngoài các scope bean tiêu chuẩn (singleton, prototype), Web Application Context cung cấp thêm 4 scope đặc thù cho web:

1.  **Request Scope (`@RequestScope`)**: Một instance bean mới sẽ được tạo ra cho mỗi một yêu cầu HTTP. Bean này chỉ tồn tại trong suốt vòng đời của yêu cầu đó.
2.  **Session Scope (`@SessionScope`)**: Một instance bean mới sẽ được tạo ra cho mỗi một phiên làm việc (HTTP Session). Bean này tồn tại cho đến khi session hết hạn hoặc bị hủy.
3.  **Application Scope (`@ApplicationScope`)**: Chỉ có một instance bean duy nhất được tạo ra cho toàn bộ vòng đời của `ServletContext` (tức là toàn bộ ứng dụng web). Nó tương tự singleton nhưng ở cấp độ `ServletContext`.
4.  **WebSocket Scope**: Bean được tạo ra cho vòng đời của một phiên WebSocket.

### Ví dụ Code

Ví dụ về một bean có phạm vi `request`:

```java
import org.springframework.stereotype.Component;
import org.springframework.web.context.annotation.RequestScope;

@Component
@RequestScope
public class RequestScopedBean {
    private final String creationTime;

    public RequestScopedBean() {
        this.creationTime = java.time.LocalTime.now().toString();
        System.out.println("RequestScopedBean được tạo lúc: " + this.creationTime);
    }

    public String getCreationTime() {
        return creationTime;
    }
}
```

Mỗi khi một yêu cầu HTTP mới được xử lý, một `RequestScopedBean` mới sẽ được tạo ra.

Ví dụ về một bean có phạm vi `session` để theo dõi giỏ hàng:

```java
import org.springframework.stereotype.Component;
import org.springframework.web.context.annotation.SessionScope;
import java.util.ArrayList;
import java.util.List;

@Component
@SessionScope
public class ShoppingCart {
    private final List<String> items = new ArrayList<>();

    public void addItem(String item) {
        items.add(item);
    }

    public List<String> getItems() {
        return items;
    }

    public ShoppingCart() {
        System.out.println("Một ShoppingCart mới đã được tạo cho session này.");
    }
}
```

`ShoppingCart` này sẽ được duy trì cho một người dùng cụ thể trong suốt phiên làm việc của họ.

-----

## 4\. Annotation @Controller được dùng để làm gì?

Annotation **`@Controller`** được dùng để đánh dấu một lớp Java là một **Controller** trong kiến trúc MVC của Spring.

Khi Spring quét các lớp trong ứng dụng (classpath scanning), nó sẽ phát hiện các lớp được đánh dấu bằng `@Controller` và đăng ký chúng như những bean có khả năng xử lý các yêu cầu web.

`@Controller` là một chuyên biệt hóa của annotation `@Component`, vì vậy nó cũng làm cho lớp trở thành một bean được quản lý bởi Spring container.

Thông thường, các phương thức bên trong một lớp `@Controller` sẽ được chú thích thêm bằng các annotation ánh xạ yêu cầu như `@RequestMapping`, `@GetMapping`, `@PostMapping`, v.v., để chỉ định yêu cầu nào chúng sẽ xử lý.

### Ví dụ Code

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

// Đánh dấu lớp này là một Controller
@Controller
public class HomeController {

    // Phương thức này sẽ xử lý các yêu cầu GET đến trang chủ ("/")
    @GetMapping("/")
    @ResponseBody // Trả về chuỗi "Welcome!" trực tiếp trong response body
    public String home() {
        return "Welcome to the Home Page!";
    }
}
```

-----

## 5\. Một yêu cầu đến được ánh xạ tới một controller và một phương thức như thế nào?

Quá trình ánh xạ một yêu cầu đến một phương thức controller cụ thể được điều phối bởi **`DispatcherServlet`** và thực hiện chủ yếu bởi **`HandlerMapping`**.

Các bước diễn ra như sau:

1.  **Yêu cầu đến `DispatcherServlet`**: Mọi yêu cầu đều đi qua `DispatcherServlet` trước tiên.
2.  **`DispatcherServlet` hỏi `HandlerMapping`**: `DispatcherServlet` sẽ duyệt qua một danh sách các `HandlerMapping` đã được đăng ký để tìm ra một "handler" (thường là một phương thức controller) phù hợp với yêu cầu hiện tại.
3.  **`HandlerMapping` tìm handler**: `HandlerMapping` sẽ kiểm tra các controller trong ứng dụng. Nó tìm kiếm các annotation như **`@RequestMapping`** (và các biến thể của nó như `@GetMapping`, `@PostMapping`,...) trên các lớp và phương thức.
4.  **Đối chiếu tiêu chí**: Ánh xạ được quyết định dựa trên nhiều tiêu chí được định nghĩa trong `@RequestMapping`:
    * `path` (hoặc `value`): Đường dẫn URI của yêu cầu (ví dụ: `/users/{id}`).
    * `method`: Phương thức HTTP (GET, POST, PUT, DELETE, v.v.).
    * `params`: Các tham số phải có (hoặc không có) trong yêu cầu (ví dụ: `myParam=myValue`).
    * `headers`: Các header phải có (hoặc không có) trong yêu cầu (ví dụ: `Content-Type=application/json`).
    * `consumes`: Loại media type mà phương thức có thể tiêu thụ (ví dụ: `application/json`).
    * `produces`: Loại media type mà phương thức sẽ tạo ra (ví dụ: `application/xml`).

Khi một phương thức controller khớp với tất cả các tiêu chí của yêu cầu, `HandlerMapping` sẽ trả về phương thức đó cho `DispatcherServlet`, và sau đó `DispatcherServlet` sẽ thực thi nó.

### Ví dụ Code

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/api/orders") // Ánh xạ ở cấp lớp, áp dụng cho tất cả các phương thức
public class OrderController {

    // Ánh xạ cho: GET /api/orders/{orderId}
    @GetMapping("/{orderId}")
    @ResponseBody
    public String getOrderById(@PathVariable String orderId) {
        return "Details for order: " + orderId;
    }

    // Ánh xạ cho: POST /api/orders
    // Chỉ chấp nhận các yêu cầu có header Content-Type là application/json
    @PostMapping(consumes = "application/json")
    @ResponseBody
    public String createOrder() {
        return "Order created successfully!";
    }

    // Ánh xạ cho: GET /api/orders/search
    // Yêu cầu phải có tham số 'status'
    @GetMapping(path = "/search", params = "status")
    @ResponseBody
    public String searchOrdersByStatus(@RequestParam String status) {
        return "Searching for orders with status: " + status;
    }
}
```

-----

## 6\. Sự khác biệt giữa @RequestMapping và @GetMapping là gì?

Sự khác biệt chính là về tính chuyên dụng và sự rõ ràng:

* **`@RequestMapping`**: Là một annotation ánh xạ chung chung, có thể được sử dụng để xử lý **bất kỳ phương thức HTTP nào** (GET, POST, PUT, DELETE, v.v.). Bạn có thể chỉ định phương thức HTTP thông qua thuộc tính `method`. Nếu không chỉ định, nó sẽ khớp với tất cả các phương thức HTTP.

* **`@GetMapping`**: Là một annotation chuyên dụng, nó là một lối tắt cho **`@RequestMapping(method = RequestMethod.GET)`**. Nó chỉ dùng để xử lý các yêu cầu HTTP GET.

**Tại sao nên dùng `@GetMapping`?**

* **Rõ ràng và ngắn gọn**: Nhìn vào `@GetMapping` là biết ngay phương thức này dùng để xử lý yêu cầu GET, giúp code dễ đọc hơn.
* **Ít lỗi hơn**: Giảm khả năng quên chỉ định `method` trong `@RequestMapping`.

Tương tự, Spring cung cấp các annotation chuyên dụng khác cho các phương thức HTTP phổ biến:

* `@PostMapping` (cho POST)
* `@PutMapping` (cho PUT)
* `@DeleteMapping` (cho DELETE)
* `@PatchMapping` (cho PATCH)

### Ví dụ Code

Hai định nghĩa phương thức sau đây là **tương đương**:

Sử dụng **`@RequestMapping`**:

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProductController {

    @RequestMapping(path = "/products", method = RequestMethod.GET)
    public String listProducts() {
        return "List of all products";
    }
}
```

Sử dụng **`@GetMapping`** (ngắn gọn và được khuyên dùng):

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProductController {

    @GetMapping("/products")
    public String listProducts() {
        return "List of all products";
    }
}
```

-----

## 7\. @RequestParam được dùng để làm gì?

Annotation **`@RequestParam`** được sử dụng để trích xuất và ràng buộc các **tham số của một yêu cầu web (request parameters)** vào các tham số của phương thức trong controller.

Các tham số yêu cầu này có thể đến từ:

* **Chuỗi truy vấn (Query string)** trong URL (ví dụ: `/search?q=spring&page=1`).
* **Dữ liệu từ một form (Form data)** được gửi đi bằng phương thức POST với `content-type` là `application/x-www-form-urlencoded`.

**Các thuộc tính quan trọng của `@RequestParam`:**

* `name` (hoặc `value`): Tên của tham số trong yêu cầu cần lấy giá trị (ví dụ: `q`). Nếu tên tham số của phương thức trùng với tên tham số yêu cầu, bạn có thể bỏ qua thuộc tính này.
* `required`: Xác định xem tham số có bắt buộc hay không (mặc định là `true`). Nếu `required=true` và tham số bị thiếu, Spring sẽ ném ra một exception. Nếu `required=false`, tham số của phương thức sẽ nhận giá trị `null` nếu nó không có mặt.
* `defaultValue`: Cung cấp một giá trị mặc định cho tham số nếu nó không được tìm thấy trong yêu cầu. Sử dụng `defaultValue` ngầm định rằng `required=false`.

### Ví dụ Code

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import java.util.Optional;

@Controller
public class SearchController {

    // URL ví dụ: /articles?topic=java&page=2
    @GetMapping("/articles")
    @ResponseBody
    public String searchArticles(
            // Tham số 'topic' là bắt buộc
            @RequestParam("topic") String topic,

            // Tham số 'page' là không bắt buộc, mặc định là "1"
            @RequestParam(name = "page", required = false, defaultValue = "1") int pageNumber,
            
            // Sử dụng Optional<String> cho tham số không bắt buộc 'author'
            @RequestParam(name = "author") Optional<String> author) {

        String authorName = author.orElse("any author"); // Nếu author không có, dùng giá trị mặc định

        return String.format("Searching for articles on topic '%s', on page %d, by %s.",
                topic, pageNumber, authorName);
    }
}
```

-----

## 8\. Sự khác biệt giữa @RequestParam và @PathVariable là gì?

Cả hai annotation này đều dùng để trích xuất dữ liệu từ URL, nhưng chúng lấy dữ liệu từ những phần khác nhau của URL.

* **`@PathVariable`**: Dùng để trích xuất giá trị từ các **biến trong đường dẫn URI (URI template variables)**. Các biến này là một phần của cấu trúc đường dẫn và thường được đặt trong dấu ngoặc nhọn `{}` trong annotation ánh xạ. Nó phù hợp để xác định một tài nguyên cụ thể.

* **`@RequestParam`**: Dùng để trích xuất giá trị từ các **tham số trong chuỗi truy vấn (query parameters)**, là các cặp `key=value` xuất hiện sau dấu `?` trong URL. Nó phù hợp cho việc lọc, sắp xếp hoặc phân trang dữ liệu.

| Tiêu chí       | `@PathVariable`                | `@RequestParam`                  |
| :------------- | :----------------------------- | :------------------------------- |
| **Nguồn dữ liệu** | Phần đường dẫn của URL         | Phần chuỗi truy vấn của URL      |
| **Ví dụ URL** | `/users/**`**`123`** | `/users?`**`id=123`** |
| **Mục đích** | Truy cập một tài nguyên cụ thể | Lọc, sắp xếp, phân trang         |
| **Giá trị mặc định** | Không hỗ trợ thuộc tính `defaultValue` | Hỗ trợ thuộc tính `defaultValue` |

### Ví dụ Code

Sử dụng **`@PathVariable`** để lấy một người dùng cụ thể bằng ID:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {
    
    // Xử lý yêu cầu như: /users/45
    // Giá trị "45" sẽ được gán cho biến userId
    @GetMapping("/users/{id}")
    public String getUserById(@PathVariable("id") long userId) {
        return "Fetching details for user with ID: " + userId;
    }
}
```

Sử dụng **`@RequestParam`** để tìm kiếm người dùng theo trạng thái:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserSearchController {

    // Xử lý yêu cầu như: /users/search?status=active&sort=name_asc
    // Giá trị "active" sẽ được gán cho userStatus
    @GetMapping("/users/search")
    public String searchUsers(
            @RequestParam("status") String userStatus,
            @RequestParam(required = false) String sort) {
        if (sort != null) {
            return String.format("Searching for users with status '%s' and sorting by '%s'", userStatus, sort);
        }
        return "Searching for users with status: " + userStatus;
    }
}
```

-----

## 9\. Một số kiểu tham số cho một phương thức controller là gì?

Các phương thức trong controller của Spring rất linh hoạt và có thể chấp nhận nhiều kiểu tham số khác nhau. Spring sẽ tự động "tiêm" (inject) các đối tượng cần thiết vào làm tham số khi phương thức được gọi. Dưới đây là một số kiểu phổ biến:

* **Các đối tượng liên quan đến Servlet API**:
    * `javax.servlet.ServletRequest` / `HttpServletRequest`: Cung cấp quyền truy cập trực tiếp vào yêu cầu HTTP.
    * `javax.servlet.ServletResponse` / `HttpServletResponse`: Cung cấp quyền truy cập trực tiếp vào phản hồi HTTP, hữu ích để thiết lập header hoặc cookie.
    * `javax.servlet.http.HttpSession`: Cho phép truy cập vào session của người dùng.
* **Các đối tượng của Spring**:
    * `WebRequest` / `NativeWebRequest`: Một giao diện của Spring trừu tượng hóa `ServletRequest` và `PortletRequest`.
    * `org.springframework.ui.Model` / `ModelMap`: Một đối tượng để bạn thêm các thuộc tính (dữ liệu) vào đó, dữ liệu này sẽ được truyền cho View để hiển thị.
    * `RedirectAttributes`: Dùng để thêm các thuộc tính cho một yêu cầu chuyển hướng (redirect), đặc biệt là flash attributes (tồn tại qua một lần redirect).
    * `Errors` / `BindingResult`: Phải được đặt ngay sau một tham số được đánh dấu bằng `@ModelAttribute` hoặc `@RequestBody` để chứa kết quả của việc ràng buộc dữ liệu và kiểm tra hợp lệ (validation).
    * `SessionStatus`: Cho phép đánh dấu session đã hoàn thành, thường dùng để xóa các thuộc tính session được quản lý bởi `@SessionAttributes`.
* **Dữ liệu yêu cầu**:
    * `java.security.Principal`: Chứa thông tin về người dùng đã được xác thực (nếu sử dụng Spring Security).
    * `HttpMethod`: Chứa phương thức HTTP của yêu cầu (GET, POST, v.v.).
    * `java.util.Locale`: Chứa thông tin về ngôn ngữ của người dùng.
    * `HttpEntity<B>`: Một đối tượng chứa cả header và body của yêu cầu.
* **Truy cập vào Body của yêu cầu/phản hồi**:
    * `java.io.InputStream` / `java.io.Reader`: Để đọc body của yêu cầu dưới dạng luồng dữ liệu thô.
    * `java.io.OutputStream` / `java.io.Writer`: Để ghi body của phản hồi dưới dạng luồng dữ liệu thô.

### Ví dụ Code

Một phương thức controller sử dụng nhiều loại tham số khác nhau:

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import javax.servlet.http.HttpServletRequest;
import java.security.Principal;
import java.util.Locale;

@Controller
public class AdvancedController {

    @GetMapping("/dashboard")
    public String showDashboard(
            Model model,                 // Để truyền dữ liệu tới View
            Principal principal,         // Để lấy thông tin người dùng đã đăng nhập
            HttpServletRequest request,  // Để lấy thông tin chi tiết của request
            Locale locale,               // Để lấy ngôn ngữ của người dùng
            @RequestHeader("User-Agent") String userAgent // Để lấy một header cụ thể
    ) {
        // Lấy tên người dùng từ Principal (do Spring Security cung cấp)
        String username = (principal != null) ? principal.getName() : "Guest";

        // Thêm dữ liệu vào Model
        model.addAttribute("username", username);
        model.addAttribute("locale", locale.getDisplayLanguage());
        model.addAttribute("userAgent", userAgent);
        
        // Lấy địa chỉ IP từ HttpServletRequest
        String clientIp = request.getRemoteAddr();
        model.addAttribute("clientIp", clientIp);

        // Trả về tên View
        return "dashboardView";
    }
}
```

-----

## 10\. Những annotation nào khác bạn có thể sử dụng trên một tham số của phương thức controller?

Ngoài `@RequestParam` và `@PathVariable`, có nhiều annotation khác rất hữu ích để sử dụng trên các tham số của phương thức controller, giúp việc trích xuất dữ liệu từ yêu cầu trở nên dễ dàng và rõ ràng hơn.

* **`@RequestBody`**: Ràng buộc toàn bộ **body của yêu cầu HTTP** vào một đối tượng. Spring sẽ sử dụng các `HttpMessageConverter` để chuyển đổi body (ví dụ từ JSON hoặc XML) thành một đối tượng Java. Thường dùng trong các API RESTful.

* **`@RequestHeader`**: Trích xuất giá trị của một **header cụ thể** trong yêu cầu HTTP (ví dụ: `User-Agent`, `Accept-Language`).

* **`@CookieValue`**: Trích xuất giá trị của một **cookie HTTP**.

* **`@ModelAttribute`**: Ràng buộc các tham số của yêu cầu (từ query string hoặc form data) vào các thuộc tính của một đối tượng. Nó cũng có thể được dùng để thêm các thuộc tính chung vào Model cho tất cả các yêu cầu trong controller.

* **`@MatrixVariable`**: Trích xuất các biến từ các đoạn URI (URI segments), là một tính năng ít phổ biến hơn cho phép các tham số được đặt trong chính đường dẫn. Ví dụ: `/cars;color=red;year=2023`.

* **`@RequestPart`**: Dùng để truy cập một phần (part) của một yêu cầu `multipart/form-data`, thường được sử dụng khi **tải lên tệp (file upload)**.

* **`@SessionAttribute`**: Ràng buộc một thuộc tính đã tồn tại trong session vào một tham số của phương thức.

### Ví dụ Code

Một ví dụ về API RESTful sử dụng nhiều annotation khác nhau:

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/products")
public class ProductApiController {

    // Ví dụ về một yêu cầu POST để tạo sản phẩm mới
    // URL: POST /api/products
    @PostMapping
    public ResponseEntity<Product> createProduct(
            // Toàn bộ body của yêu cầu (dạng JSON) sẽ được chuyển thành đối tượng Product
            @RequestBody Product newProduct,

            // Lấy giá trị của header "X-Client-ID"
            @RequestHeader("X-Client-ID") String clientId,

            // Lấy giá trị của cookie "session-token"
            @CookieValue(name = "session-token", required = false) String sessionToken
    ) {
        System.out.println("Request from client: " + clientId);
        System.out.println("Received new product: " + newProduct.getName());

        if (sessionToken != null) {
            System.out.println("User has session token.");
        }

        // Giả lập lưu sản phẩm và trả về
        newProduct.setId(1L); // Gán ID sau khi lưu
        return ResponseEntity.ok(newProduct);
    }
    
    // Một lớp giả lập cho Product
    private static class Product {
        private Long id;
        private String name;
        private double price;

        // Getters and Setters
        public Long getId() { return id; }
        public void setId(Long id) { this.id = id; }
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        public double getPrice() { return price; }
        public void setPrice(double price) { this.price = price; }
    }
}
```

Để gọi API này, bạn có thể dùng một công cụ như `curl`:

```bash
curl -X POST http://localhost:8080/api/products \
-H "Content-Type: application/json" \
-H "X-Client-ID: my-app-123" \
-b "session-token=abcdef123456" \
-d '{"name": "Laptop Pro", "price": 1200.50}'
```

-----

## 11\. Một số kiểu trả về hợp lệ của một phương thức controller là gì?

Một phương thức controller trong Spring MVC có thể trả về nhiều kiểu dữ liệu khác nhau, tùy thuộc vào mục đích của bạn là trả về một API RESTful, render một trang HTML hay xử lý bất đồng bộ.

### 1\. Trả về Dữ liệu (Thường dùng cho REST APIs)

* **`@ResponseBody`**: Annotation này có thể được dùng trên một phương thức để chỉ ra rằng giá trị trả về của phương thức đó sẽ được ghi trực tiếp vào body của response HTTP. Spring sẽ sử dụng các `HttpMessageConverter` để chuyển đổi đối tượng trả về (ví dụ: một đối tượng Java) thành một định dạng khác (ví dụ: JSON).
* **`ResponseEntity<B>`**: Cho phép kiểm soát toàn bộ response HTTP, bao gồm cả **HTTP status code**, **headers**, và **body**. Đây là cách làm rất linh hoạt và được ưa chuộng khi xây dựng các API REST.
* **`HttpEntity<B>`**: Tương tự như `ResponseEntity` nhưng không cho phép thiết lập status code.
* **`HttpHeaders`**: Chỉ trả về các header trong response mà không có body.

#### Ví dụ Code (REST API)

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserApiController {

    // Sử dụng @ResponseBody (ngầm định trong @RestController)
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        // Trả về một đối tượng User, Spring sẽ tự động chuyển nó thành JSON
        return new User(id, "John Doe");
    }

    // Sử dụng ResponseEntity để kiểm soát hoàn toàn response
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        // Giả lập việc lưu user và gán ID
        user.setId(99L);
        // Trả về đối tượng user trong body, và status code là 201 CREATED
        return new ResponseEntity<>(user, HttpStatus.CREATED);
    }

    // Một lớp giả lập cho User
    public static class User {
        private Long id;
        private String name;
        // constructor, getters, setters...
        public User(Long id, String name) { this.id = id; this.name = name; }
        public Long getId() { return id; }
        public void setId(Long id) { this.id = id; }
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
    }
}
```

-----

### 2\. Trả về View (Thường dùng cho ứng dụng Web truyền thống)

* **`String`**: Trả về một chuỗi là **tên logic của view**. Spring sẽ sử dụng một `ViewResolver` để tìm file view tương ứng (ví dụ: `home.html`) để render. Dữ liệu cho view có thể được truyền thông qua tham số `Model` của phương thức.
* **`ModelAndView`**: Một đối tượng chứa cả **Model** (dữ liệu) và **View** (tên view hoặc một instance của view). Đây là cách làm tường minh để trả về cả hai cùng một lúc.
* **`View`**: Trả về một instance của một view cụ thể (ví dụ: `JstlView`, `ThymeleafView`).
* **`Map` / `Model`**: Trả về một đối tượng `Map` hoặc `Model` chứa các thuộc tính sẽ được thêm vào model. Tên view sẽ được Spring tự động suy ra dựa trên URL của yêu cầu (thông qua `RequestToViewNameTranslator`).

#### Ví dụ Code (Web Application)

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class HomeController {

    // Trả về tên view dạng String, dữ liệu được truyền qua Model
    @GetMapping("/home")
    public String homePage(Model model) {
        model.addAttribute("message", "Welcome to our website!");
        return "home"; // Sẽ tìm file home.html (hoặc .jsp, v.v.)
    }

    // Trả về một đối tượng ModelAndView
    @GetMapping("/profile")
    public ModelAndView userProfile() {
        ModelAndView mav = new ModelAndView("userProfile"); // Đặt tên view là "userProfile"
        mav.addObject("username", "JaneDoe");
        mav.addObject("email", "jane.doe@example.com");
        return mav;
    }
}
```

-----

### 3\. Các kiểu trả về khác

* **`void`**: Phương thức không trả về gì. Nó có thể tự xử lý response bằng cách ghi trực tiếp vào `HttpServletResponse`, hoặc chỉ định status code bằng `@ResponseStatus`. Trong REST API, `void` cùng với status 204 No Content thường có nghĩa là "thao tác thành công, không có nội dung gì để trả về".
* **Kiểu bất đồng bộ**: Spring hỗ trợ xử lý bất đồng bộ để giải phóng luồng của servlet container, cho phép ứng dụng xử lý nhiều yêu cầu hơn.
    * **`Callable<V>`**: Cho phép một tác vụ được thực thi trong một luồng do Spring quản lý.
    * **`DeferredResult<V>`**: Kết quả sẽ được trả về từ một luồng khác trong tương lai.
    * **`CompletableFuture<V>`**: Hỗ trợ các chuỗi xử lý bất đồng bộ phức tạp.
* **Kiểu streaming**: Dùng để gửi dữ liệu theo dòng (stream).
    * **`ResponseBodyEmitter` / `SseEmitter`**: Cho phép gửi nhiều đối tượng một cách bất đồng bộ về phía client. `SseEmitter` chuyên dùng cho Server-Sent Events.
    * **`StreamingResponseBody`**: Cho phép ghi dữ liệu bất đồng bộ vào `OutputStream` của response.

#### Ví dụ Code (void và bất đồng bộ)

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import java.util.concurrent.Callable;

@RestController
public class OtherReturnTypesController {

    // Sử dụng void và @ResponseStatus
    @DeleteMapping("/items/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT) // Trả về status 204 No Content
    public void deleteItem(@PathVariable Long id) {
        System.out.println("Deleting item with ID: " + id);
        // ... logic xóa item
    }

    // Xử lý bất đồng bộ với Callable
    @GetMapping("/async-task")
    public Callable<String> performAsyncTask() {
        System.out.println("Main thread received request. Handing over to Callable.");
        return () -> {
            Thread.sleep(2000); // Giả lập công việc tốn thời gian
            System.out.println("Callable task completed.");
            return "Async task completed successfully!";
        };
    }
}
```