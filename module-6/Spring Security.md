Dưới đây là thông tin về các câu hỏi bạn đã đưa ra, được trình bày một cách rõ ràng và dễ hiểu:

-----

## 1\. Xác thực (Authentication) và Phân quyền (Authorization) là gì? Cái nào phải có trước?

**Xác thực (Authentication)** là quá trình xác minh danh tính của người dùng, thiết bị hoặc hệ thống bên ngoài để đảm bảo họ đúng là người mà họ tự nhận. Nó trả lời cho câu hỏi: **"Bạn là ai?"**. Quá trình này thường bao gồm việc gửi thông tin định danh (Identity) và thông tin xác thực (Credential), sau đó hệ thống sẽ kiểm tra thông tin xác thực để chấp nhận hoặc từ chối danh tính đã được khai báo. Hình thức đơn giản nhất là sử dụng tên người dùng (username) và mật khẩu (password).

Spring Security hỗ trợ nhiều cơ chế xác thực như:

* Đăng nhập bằng form (Form Login)
* Xác thực cơ bản (Basic Authentication)
* OAuth 2.0
* SAML 2.0
* Xác thực qua chứng chỉ X.509

**Phân quyền (Authorization)** là quá trình xác định xem một người dùng đã được xác thực có được phép truy cập vào các tài nguyên nhất định hoặc thực hiện các hành động cụ thể trong hệ thống hay không. Nó trả lời cho câu hỏi: **"Bạn được phép làm gì?"**. Quá trình này dựa trên các quy tắc kiểm soát truy cập (access control rules) đã được định nghĩa sẵn, có thể thông qua:

* **Vai trò (Roles)**: Tập hợp các quyền hạn ở mức độ cao, ví dụ `ROLE_ADMIN`, `ROLE_STAFF`.
* **Quyền hạn (Authorities)**: Các quyền chi tiết, mức độ thấp hơn, ví dụ `READ_CUSTOMERS`, `DELETE_EMPLOYEE`.

**Xác thực (Authentication) phải được thực hiện trước phân quyền (Authorization)**. Lý do là hệ thống không thể quyết định một người dùng được phép làm gì (phân quyền) nếu chưa biết danh tính chính xác của người dùng đó (xác thực). Bạn phải biết "bạn là ai" trước khi có thể quyết định "bạn được làm gì".

-----

## 2\. Bảo mật có phải là một mối quan tâm xuyên suốt (cross-cutting concern) không? Nó được triển khai nội bộ như thế nào?

Đúng, **bảo mật là một mối quan tâm xuyên suốt (cross-cutting concern)**. Một mối quan tâm xuyên suốt là một chức năng của chương trình không liên quan trực tiếp đến logic nghiệp vụ nhưng lại được áp dụng trên toàn bộ ứng dụng và ảnh hưởng đến nhiều phần của hệ thống. Các ví dụ khác bao gồm ghi log (logging) và quản lý giao dịch (transactions).

### Cách triển khai nội bộ

Spring Security triển khai bảo mật ở hai cấp độ:

1.  **Cấp độ Web (Web Level)**: Dựa trên **Servlet Filters** để phân tích mọi yêu cầu HTTP đến hệ thống. Nó quyết định xác thực hoặc phân quyền dựa trên các quy tắc được cấu hình, chẳng hạn như chuyển hướng đến trang đăng nhập hoặc từ chối yêu cầu do thiếu quyền.

2.  **Cấp độ Phương thức (Method Security Level)**: Dựa trên **Spring AOP (Lập trình hướng khía cạnh)** để tạo proxy cho các lời gọi đối tượng. Nó đảm bảo rằng người dùng phải đáp ứng các quy tắc bảo mật (ví dụ: có một vai trò nhất định) trước khi được phép thực thi một phương thức.

Để kích hoạt bảo mật cấp độ phương thức, bạn cần sử dụng annotation `@EnableGlobalMethodSecurity` và bật hỗ trợ cho các loại annotation cụ thể như `@PreAuthorize` (prePostEnabled), `@Secured` (securedEnabled), hoặc `@RolesAllowed` (jsr250Enabled).

### Ví dụ Code

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
// Kích hoạt bảo mật cấp độ phương thức với annotation @PreAuthorize
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // Cấu hình bảo mật CẤP ĐỘ WEB
        http
            .authorizeRequests(authorize -> authorize
                .mvcMatchers("/admin/**").hasRole("ADMIN") // Yêu cầu vai trò ADMIN cho các URL /admin/**
                .anyRequest().authenticated() // Mọi yêu cầu khác đều cần xác thực
            )
            .formLogin(); // Kích hoạt form đăng nhập mặc định
        return http.build();
    }
    
    // Cấu hình người dùng trong bộ nhớ để test
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.withDefaultPasswordEncoder()
            .username("user")
            .password("password")
            .roles("USER")
            .build();
        UserDetails admin = User.withDefaultPasswordEncoder()
            .username("admin")
            .password("password")
            .roles("ADMIN")
            .build();
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;

@Service
public class ReportService {
    
    // Cấu hình bảo mật CẤP ĐỘ PHƯƠNG THỨC
    // Chỉ người dùng có vai trò 'ADMIN' mới có thể gọi phương thức này
    @PreAuthorize("hasRole('ADMIN')")
    public String generateAdminReport() {
        return "Đây là báo cáo mật dành cho admin.";
    }

    public String generatePublicReport() {
        return "Đây là báo cáo công khai.";
    }
}
```

-----

## 3\. DelegatingFilterProxy là gì?

**`DelegatingFilterProxy`** là một lớp nội bộ của Spring Framework, hoạt động như một **proxy** giữa một Servlet Filter tiêu chuẩn và một Bean do Spring quản lý (cũng triển khai giao diện Servlet Filter). Nói một cách đơn giản, nó hoạt động như một **cây cầu** nối giữa Servlet Container (như Tomcat) và Spring Application Context.

**Tại sao nó lại cần thiết?**

* Servlet Filter được quản lý bởi Servlet Container và vòng đời của chúng tách biệt với Spring.
* Các bộ lọc bảo mật của Spring (Security Filters) là các bean được quản lý bởi Spring, tận dụng được các tính năng như Dependency Injection.

`DelegatingFilterProxy` được đăng ký trong container servlet. Khi một yêu cầu đến, `DelegatingFilterProxy` sẽ bắt nó và ủy quyền (delegate) tất cả các lời gọi đến một bean filter trong Spring Context. Bean này thường có tên là `springSecurityFilterChain` và là một instance của `FilterChainProxy`.

Trong các ứng dụng Spring Boot, việc đăng ký `DelegatingFilterProxy` được thực hiện tự động bởi `SecurityFilterAutoConfiguration`.

-----

## 4\. Chuỗi bộ lọc bảo mật (security filter chain) là gì?

**Security Filter Chain** (cụ thể là `SecurityFilterChain`) là một tập hợp các bộ lọc (Filter) do Spring quản lý, chịu trách nhiệm về việc xác thực và phân quyền. Mỗi `SecurityFilterChain` được liên kết với một `RequestMatcher` để xác định xem chuỗi bộ lọc đó có nên được áp dụng cho một yêu cầu HTTP cụ thể hay không.

Ví dụ, bạn có thể có một chuỗi bộ lọc cho các URL `/api/**` và một chuỗi khác cho `/admin/**`.

Khi một yêu cầu đến, `FilterChainProxy` (bean mà `DelegatingFilterProxy` ủy quyền) sẽ tìm kiếm `SecurityFilterChain` đầu tiên khớp với yêu cầu đó. Sau khi tìm thấy, `FilterChainProxy` sẽ thực thi tuần tự các bộ lọc trong chuỗi đó.

Một chuỗi bộ lọc bảo mật của Spring thường bao gồm các bộ lọc sau:

* `SecurityContextPersistenceFilter`
* `HeaderWriterFilter`
* `CsrfFilter`
* `UsernamePasswordAuthenticationFilter`
* `SessionManagementFilter`
* `FilterSecurityInterceptor`

### Ví dụ Code

Trong các phiên bản Spring Security hiện đại, bạn định nghĩa `SecurityFilterChain` như một Bean.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class MultipleSecurityConfigs {

    @Bean
    @Order(1) // Ưu tiên chuỗi này trước
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        http
            .antMatcher("/api/**") // Chuỗi này chỉ áp dụng cho /api/**
            .authorizeRequests(authorize -> authorize
                .anyRequest().hasRole("API_USER")
            )
            .httpBasic(); // Sử dụng xác thực Basic cho API
        return http.build();
    }

    @Bean
    public SecurityFilterChain webFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests(authorize -> authorize
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(); // Sử dụng form đăng nhập cho các trang web khác
        return http.build();
    }
}
```

-----

## 5\. Security context là gì?

**`SecurityContext`** là một giao diện chứa thông tin bảo mật liên quan đến luồng thực thi (thread of execution) hiện tại. Về cơ bản, nó chứa đối tượng `Authentication`, đại diện cho người dùng đã được xác thực.

`SecurityContext` được lưu trữ trong **`SecurityContextHolder`**, đây là thành phần cốt lõi của mô hình xác thực trong Spring Security. `SecurityContextHolder` sử dụng một chiến lược lưu trữ để quyết định cách `SecurityContext` được lưu giữ. Mặc định là **`MODE_THREADLOCAL`**, nghĩa là mỗi luồng yêu cầu sẽ có `SecurityContext` riêng của nó.

Các chiến lược khác bao gồm:

* `MODE_INHERITABLETHREADLOCAL`: Cho phép các luồng con kế thừa `SecurityContext` từ luồng cha.
* `MODE_GLOBAL`: Tất cả các luồng trong JVM chia sẻ cùng một `SecurityContext`, thường dùng cho các ứng dụng desktop.

Đối tượng `Authentication` bên trong `SecurityContext` chứa các thông tin quan trọng:

* `getPrincipal()`: Danh tính của người dùng (thường là đối tượng `UserDetails`).
* `getCredentials()`: Thông tin xác thực (ví dụ: mật khẩu), thường bị xóa sau khi xác thực thành công.
* `getAuthorities()`: Tập hợp các quyền (`GrantedAuthority`) được cấp cho người dùng.

### Ví dụ Code

Bạn có thể truy cập thông tin người dùng hiện tại một cách an toàn từ bất kỳ đâu trong ứng dụng thông qua `SecurityContextHolder`.

```java
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    public String getCurrentUsername() {
        // Lấy SecurityContext từ SecurityContextHolder
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication != null && authentication.isAuthenticated()) {
            Object principal = authentication.getPrincipal();
            if (principal instanceof UserDetails) {
                return ((UserDetails) principal).getUsername();
            } else {
                return principal.toString();
            }
        }
        return "Anonymous";
    }
}
```

-----

## 6\. Mẫu `**` trong `antMatcher` hoặc `mvcMatcher` có tác dụng gì?

Mẫu `**` trong `antMatcher` và `mvcMatcher` được sử dụng để khớp với **không hoặc nhiều phân đoạn đường dẫn (path segments)** cho đến cuối cùng.

Các quy tắc khớp mẫu (wildcard) khác bao gồm:

* `?`: Khớp với một ký tự duy nhất.
* `*`: Khớp với không hoặc nhiều ký tự **bên trong một phân đoạn đường dẫn**.

**Ví dụ:** Cho URL `/departments/delete/5`:

* `/departments/delete/*` - **Khớp** (`*` khớp với `5`)
* `/departments/delete/**` - **Khớp** (`**` khớp với `/5`)
* `/**/5` - **Khớp** (`**` khớp với `/departments/delete`)
* `/*/5` - **Không khớp** (`*` chỉ khớp với một phân đoạn, ở đây là `departments`, không khớp với `departments/delete`)

### Ví dụ Code

```java
@Configuration
public class PathMatcherConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            // Cho phép truy cập công khai vào tất cả các file trong thư mục /css và /js
            .antMatchers("/css/**", "/js/**").permitAll()
            
            // Yêu cầu vai trò ADMIN cho bất kỳ URL nào trong /admin
            .antMatchers("/admin/**").hasRole("ADMIN")
            
            // Mọi yêu cầu khác đều cần xác thực
            .anyRequest().authenticated();
            
        return http.build();
    }
}
```

-----

## 7\. Tại sao việc sử dụng `mvcMatcher` được khuyến khích hơn `antMatcher`?

`mvcMatcher` được khuyến khích hơn vì nó **linh hoạt và dễ sử dụng hơn** khi viết các quy tắc bảo mật, giúp giảm thiểu sai sót.

Sự khác biệt chính và quan trọng nhất là cách chúng xử lý dấu gạch chéo (`/`) ở cuối.

Hãy xem xét quy tắc `("/employees")`:

* `mvcMatchers("/employees")`: Sẽ khớp với cả `/employees` **và** `/employees/`.
* `antMatchers("/employees")`: Sẽ khớp với `/employees` nhưng **không** khớp với `/employees/`.

Do đó, khi sử dụng `antMatchers`, rất dễ bỏ sót dấu gạch chéo cuối cùng, tạo ra một lỗ hổng bảo mật tiềm tàng cho phép kẻ tấn công bỏ qua các quy tắc bảo mật của ứng dụng. `mvcMatcher` sử dụng cơ chế khớp mẫu của Spring MVC, vốn đã được thiết kế để xử lý những trường hợp này một cách nhất quán.

### Ví dụ Code

```java
@Configuration
public class MatcherChoiceConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            // Khuyến khích dùng mvcMatchers để tránh lỗi với dấu gạch chéo cuối
            .mvcMatchers("/dashboard").hasRole("USER")
            
            // Nếu dùng antMatchers, phải cẩn thận hơn
            .antMatchers("/reports", "/reports/").hasRole("ANALYST")
            
            .anyRequest().authenticated();
            
        return http.build();
    }
}
```

-----

## 8\. Spring Security có hỗ trợ băm mật khẩu (password hashing) không? Salting là gì?

Có, Spring Security hỗ trợ mạnh mẽ việc băm mật khẩu thông qua giao diện **`PasswordEncoder`**. Mật khẩu không bao giờ được lưu trữ dưới dạng văn bản thuần túy.

Spring Security cung cấp sẵn các bộ mã hóa hiện đại và an toàn như:

* `bcrypt`
* `pbkdf2`
* `scrypt`
* `argon2`

Giao diện `PasswordEncoder` có hai phương thức chính:

* `encode(rawPassword)`: Mã hóa mật khẩu thô.
* `matches(rawPassword, encodedPassword)`: Kiểm tra xem mật khẩu thô có khớp với mật khẩu đã được mã hóa hay không. Quá trình này không giải mã mật khẩu mà mã hóa lại mật khẩu thô và so sánh kết quả.

### Salting là gì?

**Salting** là một cơ chế bảo mật được sử dụng để chống lại các cuộc tấn công đảo ngược hàm băm bằng cách sử dụng các bảng tính toán trước như Rainbow Tables.

Ý tưởng là trước khi băm một mật khẩu, một chuỗi byte ngẫu nhiên gọi là **salt** sẽ được thêm vào mật khẩu đó. Salt này sau đó được lưu trữ cùng với kết quả băm trong cơ sở dữ liệu. Điều này đảm bảo rằng ngay cả khi hai người dùng có cùng một mật khẩu, giá trị băm của họ trong cơ sở dữ liệu vẫn sẽ khác nhau do có salt khác nhau.

Khi xác minh mật khẩu, hệ thống sẽ lấy salt đã lưu, kết hợp nó với mật khẩu thô mà người dùng cung cấp, băm lại và so sánh kết quả với giá trị băm đã lưu. Các bộ mã hóa hiện đại như `BCryptPasswordEncoder` đã tự động tích hợp việc tạo và quản lý salt.

### Ví dụ Code

Bạn cần định nghĩa một bean `PasswordEncoder` trong cấu hình của mình.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class PasswordEncoderConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        // BCrypt là một lựa chọn mạnh mẽ và được khuyến nghị
        return new BCryptPasswordEncoder();
    }
}
```

Spring Security sẽ tự động sử dụng bean này để mã hóa và xác minh mật khẩu.

-----

## 9\. Tại sao bạn cần bảo mật cấp độ phương thức (method security)?

Bảo mật cấp độ phương thức là cần thiết khi bạn cần các **quy tắc bảo mật chi tiết và chặt chẽ hơn** cho ứng dụng.

Trong nhiều trường hợp, việc chỉ bảo vệ dựa trên các mẫu URL (bảo mật cấp độ web) là không đủ. Ví dụ, nhiều hành động khác nhau có thể cùng chia sẻ một URL (ví dụ `POST /users`), nhưng chỉ admin mới được tạo người dùng mới, còn người dùng thông thường chỉ được cập nhật thông tin của chính họ.

Bảo mật cấp độ phương thức giúp áp dụng các quy tắc này trực tiếp lên **tầng dịch vụ (service layer)** của ứng dụng, nơi chứa logic nghiệp vụ. Điều này tạo ra một lớp bảo vệ thứ hai, sâu hơn, đảm bảo rằng ngay cả khi lớp web bị bỏ qua, logic nghiệp vụ vẫn được bảo vệ.

Spring hỗ trợ nhiều annotation cho bảo mật cấp độ phương thức:

* `@PreAuthorize`, `@PostAuthorize`, `@PreFilter`, `@PostFilter`
* `@Secured`
* `@RolesAllowed` (tiêu chuẩn JSR-250)

### Ví dụ Code

Để sử dụng, bạn cần kích hoạt nó trong cấu hình bảo mật.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true) // Kích hoạt @PreAuthorize, @PostAuthorize
public class MethodSecurityConfig {
    // Các cấu hình khác
}
```

Sau đó, áp dụng lên các phương thức trong tầng service.

```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;

@Service
public class AccountService {

    // Chỉ người dùng có vai trò 'ADMIN' mới có thể đóng tài khoản bất kỳ
    @PreAuthorize("hasRole('ADMIN')")
    public void closeAccount(Long accountId) {
        // ... logic đóng tài khoản
    }

    // Người dùng chỉ có thể xem chi tiết tài khoản của chính họ
    // '#username' tham chiếu đến tham số 'username' của phương thức
    // 'authentication.name' là username của người dùng đang đăng nhập
    @PreAuthorize("#username == authentication.name")
    public AccountDetails viewAccountDetails(String username) {
        // ... logic lấy chi tiết tài khoản
        return new AccountDetails();
    }
}
```

-----

## 10\. Annotation `@PreAuthorize` và `@RolesAllowed` làm gì? Sự khác biệt giữa chúng là gì?

Cả `@PreAuthorize` và `@RolesAllowed` đều là các annotation thuộc mô hình bảo mật cấp độ phương thức của Spring Security. Cả hai đều được đánh giá **trước khi** một phương thức được thực thi để kiểm tra xem người dùng có được phép gọi phương thức đó hay không.

### @RolesAllowed

* Là một phần của tiêu chuẩn JSR-250.
* Nó cho phép bạn chỉ định một **danh sách các vai trò (roles)** mà người dùng đã xác thực phải có để được phép thực thi phương thức.
* Cách sử dụng đơn giản, chỉ để kiểm tra vai trò.
* Cần được kích hoạt bằng `@EnableGlobalMethodSecurity(jsr250Enabled = true)`.

### @PreAuthorize

* Cho phép bạn chỉ định các điều kiện bảo mật bằng cách sử dụng **Ngôn ngữ Biểu thức Spring (SpEL)**.
* Mạnh mẽ và linh hoạt hơn rất nhiều, cho phép viết các logic phức tạp, ví dụ:
    * `hasRole('ROLE_ADMIN')` hoặc `hasAnyRole('ROLE_ADMIN', 'ROLE_STAFF')`
    * `isAuthenticated()`
    * Kiểm tra tham số của phương thức: `hasPermission(#invoice, 'write')`
    * So sánh danh tính người dùng: `#username == authentication.name`
* Cần được kích hoạt bằng `@EnableGlobalMethodSecurity(prePostEnabled = true)`.

**Sự khác biệt chính**: `@RolesAllowed` chỉ dùng để kiểm tra vai trò một cách đơn giản, trong khi `@PreAuthorize` cho phép bạn viết các quy tắc bảo mật phức tạp và động bằng cách sử dụng SpEL.

### Ví dụ Code

```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;
import javax.annotation.security.RolesAllowed;

@Service
public class DocumentService {

    // Chỉ người dùng có vai trò 'ADMIN' hoặc 'EDITOR' mới được phép
    @RolesAllowed({"ADMIN", "EDITOR"})
    public void editDocument(Long docId) {
        // ... logic chỉnh sửa tài liệu
    }

    // Chỉ chủ sở hữu của tài liệu (owner) mới có thể xóa nó
    // SpEL được dùng để gọi một bean khác (@permissionService) để kiểm tra
    @PreAuthorize("@permissionService.isOwner(authentication, #docId)")
    public void deleteDocument(Long docId) {
        // ... logic xóa tài liệu
    }
}
```

-----

## 11\. Annotation `@PreAuthorized` và `@RolesAllowed` được triển khai như thế nào?

Các annotation `@PreAuthorized` và `@RolesAllowed` được triển khai bằng cách sử dụng **Spring AOP (Lập trình hướng khía cạnh)** và các **`AccessDecisionVoter`**.

Về cơ bản, luồng hoạt động diễn ra như sau:

1.  **Spring AOP** tạo ra một proxy xung quanh các bean được bảo vệ.
2.  Khi một phương thức được bảo vệ (ví dụ, có annotation `@PreAuthorize`) được gọi, một `MethodSecurityInterceptor` sẽ chặn lời gọi này.
3.  `MethodSecurityInterceptor` sau đó gọi một `AccessDecisionManager`.
4.  `AccessDecisionManager` sẽ hỏi một hoặc nhiều `AccessDecisionVoter` để "bỏ phiếu" quyết định xem quyền truy cập có được cấp hay không.

Mỗi loại annotation được xử lý bởi một `AccessDecisionVoter` cụ thể:

* **`@RolesAllowed`** được xử lý bởi `Jsr250Voter`.
* **`@PreAuthorized`** được xử lý bởi `PreInvocationAuthorizationAdviceVoter`.

Cơ chế này cho phép Spring Security áp dụng các quy tắc bảo mật một cách linh hoạt trước khi thực thi logic nghiệp vụ của phương thức.

### Ví dụ Code

Để cơ chế trên hoạt động, bạn cần kích hoạt nó trong file cấu hình Spring Security.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;

@Configuration
@EnableGlobalMethodSecurity(
    prePostEnabled = true, // Kích hoạt @PreAuthorize và @PostAuthorize
    jsr250Enabled = true   // Kích hoạt @RolesAllowed
)
public class MethodSecurityConfig {
    // Đây là nơi bạn kích hoạt cơ chế AOP cho bảo mật phương thức.
    // Spring Security sẽ tự động cấu hình các interceptor và voter cần thiết.
}
```

Sau đó bạn có thể sử dụng các annotation trong các lớp service của mình.

```java
import org.springframework.stereotype.Service;
import org.springframework.security.access.prepost.PreAuthorize;
import javax.annotation.security.RolesAllowed;

@Service
public class OrderService {

    // Được triển khai bởi Jsr250Voter
    @RolesAllowed("ROLE_ADMIN")
    public void cancelOrder(Long orderId) {
        System.out.println("Đơn hàng " + orderId + " đã bị hủy bởi admin.");
        // Logic hủy đơn hàng
    }

    // Được triển khai bởi PreInvocationAuthorizationAdviceVoter
    // Chỉ người dùng có quyền 'write' trên đối tượng 'order' mới được thực thi
    @PreAuthorize("hasPermission(#order, 'write')")
    public void updateOrder(Order order) {
        System.out.println("Đang cập nhật đơn hàng: " + order.getId());
        // Logic cập nhật đơn hàng
    }
}
```

-----

## 12\. Trong annotation bảo mật nào bạn được phép sử dụng SpEL?

Bạn được phép sử dụng **SpEL (Spring Expression Language)** trong các annotation bảo mật sau đây của Spring Security:

* `@PreAuthorize`
* `@PostAuthorize`
* `@PreFilter`
* `@PostFilter`

Sự khác biệt chính giữa chúng là:

* **`@PreAuthorize` / `@PostAuthorize`**: Được sử dụng để tạo biểu thức kiểm tra xem một phương thức có thể được thực thi hay không. `@PreAuthorize` kiểm tra trước khi thực thi, còn `@PostAuthorize` kiểm tra sau khi thực thi (và có thể truy cập vào kết quả trả về của phương thức).
* **`@PreFilter` / `@PostFilter`**: Được sử dụng để lọc các đối số hoặc kết quả trả về dạng Collection hoặc mảng dựa trên các quy tắc bảo mật. `@PreFilter` lọc đầu vào, `@PostFilter` lọc đầu ra.

### Ví dụ Code

Giả sử chúng ta có một lớp `TaskService` để quản lý các công việc.

```java
import org.springframework.security.access.prepost.PostAuthorize;
import org.springframework.security.access.prepost.PostFilter;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.access.prepost.PreFilter;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class TaskService {

    // @PreAuthorize: Chỉ người dùng có vai trò 'MANAGER' mới có thể tạo task.
    @PreAuthorize("hasRole('ROLE_MANAGER')")
    public Task createTask(Task task) {
        System.out.println("Creating task: " + task.getName());
        // ... logic lưu task
        return task;
    }

    // @PostAuthorize: Người dùng chỉ có thể xem task nếu họ là người được giao (assignee)
    // 'returnObject' là từ khóa SpEL tham chiếu đến đối tượng Task được trả về
    @PostAuthorize("returnObject.assignee == authentication.name")
    public Task getTaskById(Long id) {
        // ... logic lấy task từ DB
        // Giả sử task này được giao cho 'user1'
        return new Task(id, "Review Document", "user1");
    }

    // @PreFilter: Lọc danh sách đầu vào, chỉ xử lý các task được giao cho người dùng hiện tại
    // 'filterObject' là từ khóa SpEL tham chiếu đến từng phần tử trong collection
    @PreFilter("filterObject.assignee == authentication.name")
    public void completeTasks(List<Task> tasks) {
        System.out.println("Completing " + tasks.size() + " tasks for current user.");
        // ... logic hoàn thành các task đã được lọc
    }


    // @PostFilter: Trả về danh sách tất cả các task, nhưng lọc ra chỉ những task
    // mà người dùng hiện tại được phép xem (là assignee).
    @PostFilter("filterObject.assignee == authentication.name")
    public List<Task> getAllTasks() {
        // ... logic lấy tất cả các task từ DB
        // Giả sử danh sách này có nhiều task với các assignee khác nhau
        return List.of(
                new Task(1L, "Task 1", "user1"),
                new Task(2L, "Task 2", "user2"),
                new Task(3L, "Task 3", "user1")
        );
    }

    // Lớp giả lập cho Task
    public static class Task {
        private Long id;
        private String name;
        private String assignee; // Tên người được giao
        // constructor, getters, setters
        public Task(Long id, String name, String assignee) {
            this.id = id; this.name = name; this.assignee = assignee;
        }
        public Long getId() { return id; }
        public String getName() { return name; }
        public String getAssignee() { return assignee; }
    }
}
```