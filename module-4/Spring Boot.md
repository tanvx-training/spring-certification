-----

## Giới thiệu về Spring Boot

Phần này sẽ giải thích các khái niệm cơ bản, ưu điểm và cách hoạt động của Spring Boot.

### Câu hỏi 01: Spring Boot là gì?

**Spring Boot** là một framework được xây dựng trên nền tảng của Spring Framework, giúp cho việc tạo các ứng dụng Spring độc lập (stand-alone), sẵn sàng cho môi trường sản xuất (production-grade) trở nên **dễ dàng và nhanh chóng hơn**.

Nó không phải là một sự thay thế cho Spring Framework mà là một cách để sử dụng Spring một cách hiệu quả hơn. Spring Boot loại bỏ phần lớn công việc cấu hình thủ công mà trước đây các nhà phát triển phải làm khi thiết lập một dự án Spring.

-----

### Câu hỏi 02: Các ưu điểm của việc sử dụng Spring Boot là gì?

Sử dụng Spring Boot mang lại rất nhiều lợi ích, giúp tăng tốc độ phát triển và đơn giản hóa việc vận hành:

* **Tự động cấu hình (Auto-configuration)**: Tự động cấu hình ứng dụng của bạn dựa trên các dependency (thư viện) mà bạn thêm vào. Ví dụ, nếu phát hiện `spring-webmvc` trên classpath, nó sẽ tự động cấu hình `DispatcherServlet`.
* **"Opinionated" (Có chính kiến)**: Cung cấp một bộ cấu hình mặc định "hợp lý" cho hầu hết các trường hợp, giúp bạn bắt đầu nhanh chóng mà không cần suy nghĩ nhiều về cấu hình.
* **Starter POMs**: Đơn giản hóa việc quản lý dependency trong Maven/Gradle bằng cách cung cấp các "gói" dependency đã được kiểm thử và tương thích với nhau.
* **Server nhúng (Embedded Server)**: Dễ dàng đóng gói ứng dụng của bạn với một server nhúng (như Tomcat, Jetty, hoặc Undertow), cho phép bạn chạy ứng dụng như một file JAR thực thi độc lập (`java -jar myapp.jar`).
* **Các tính năng sẵn sàng cho sản xuất (Production-ready features)**: Tích hợp sẵn các tính năng như metrics (số liệu), health checks (kiểm tra sức khỏe), và externalized configuration (cấu hình bên ngoài).

-----

### Câu hỏi 03: Tại sao nó được gọi là "opinionated" (có chính kiến)?

Spring Boot được gọi là "opinionated" vì nó đưa ra những **"ý kiến"** về cách tốt nhất để cấu hình và xây dựng một ứng dụng. Thay vì yêu cầu bạn phải đưa ra mọi quyết định cấu hình, Spring Boot sẽ tự quyết định giúp bạn dựa trên các quy ước và thực tiễn tốt nhất.

Ví dụ, nếu bạn thêm `spring-boot-starter-web` vào dự án, Spring Boot sẽ "cho rằng" (opines) bạn muốn xây dựng một ứng dụng web và sẽ tự động cấu hình:

* Một máy chủ Tomcat nhúng.
* `DispatcherServlet` của Spring MVC.
* Các bộ chuyển đổi JSON (như Jackson).

Tuy nhiên, "chính kiến" của Spring Boot không phải là độc đoán. Nếu bạn không đồng ý với các lựa chọn mặc định, bạn hoàn toàn có thể **ghi đè** chúng và tùy chỉnh theo ý muốn.

-----

### Câu hỏi 04: Những yếu tố nào ảnh hưởng đến những gì Spring Boot thiết lập?

Việc tự động cấu hình của Spring Boot bị ảnh hưởng chủ yếu bởi ba yếu tố:

1.  **Các Starter trên Classpath**: Đây là yếu tố quan trọng nhất. Spring Boot sẽ kiểm tra các thư viện (đặc biệt là các starter) có trên classpath của bạn để quyết định cần cấu hình những gì. Ví dụ, sự hiện diện của `spring-boot-starter-data-jpa` sẽ kích hoạt việc tự động cấu hình JPA và một `DataSource`.
2.  **Cấu hình của chính bạn (Your own configuration)**: Nếu bạn tự định nghĩa một bean (ví dụ: bạn tự tạo một bean `DataSource`), Spring Boot sẽ nhận ra điều đó và **sẽ không** tạo ra bean mặc định của nó nữa. Điều này cho phép bạn dễ dàng ghi đè các cấu hình mặc định.
3.  **Các thuộc tính trong `application.properties` hoặc `application.yml`**: Bạn có thể tùy chỉnh hành vi của các cấu hình tự động bằng cách thiết lập các thuộc tính trong các tệp cấu hình. Ví dụ, bạn có thể thay đổi cổng server mặc định bằng cách đặt `server.port=9090`.

-----

### Câu hỏi 05: Spring Boot Starter POM là gì? Tại sao nó hữu ích?

Một **Spring Boot Starter POM** (hoặc đơn giản là "starter") là một bộ mô tả dependency tiện lợi mà bạn có thể thêm vào dự án Maven hoặc Gradle của mình.

**Nó hữu ích vì:**

* **Quản lý dependency đơn giản**: Thay vì phải tự mình thêm hàng chục dependency riêng lẻ và lo lắng về phiên bản tương thích của chúng, bạn chỉ cần thêm một starter duy nhất. Ví dụ, chỉ cần thêm `spring-boot-starter-web` là bạn đã có tất cả những gì cần thiết cho một ứng dụng web Spring MVC (Tomcat, Spring MVC, Jackson, ...).
* **Đảm bảo tương thích**: Các starter được quản lý bởi đội ngũ Spring Boot, đảm bảo rằng tất cả các dependency bên trong chúng đều tương thích và đã được kiểm thử cùng nhau. Điều này giúp tránh các xung đột phiên bản phức tạp.
* **Khởi đầu nhanh chóng**: Cung cấp một điểm khởi đầu "một cửa" (one-stop-shop) cho tất cả các công nghệ Spring và các công nghệ liên quan mà bạn cần.

#### Ví dụ trong `pom.xml`

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

-----

### Câu hỏi 06: Spring Boot hỗ trợ cả tệp properties và YML. Bạn có sử dụng một trong hai không? Tại sao?

Việc lựa chọn giữa tệp `.properties` và `.yml` (YAML) thường phụ thuộc vào sở thích cá nhân và yêu cầu của dự án, vì Spring Boot hỗ trợ cả hai một cách hoàn hảo.

* **Tệp `.properties`**:

    * **Cú pháp**: Dạng cặp `key=value` phẳng.
    * **Ưu điểm**: Đơn giản, dễ hiểu, được hỗ trợ rộng rãi bởi các IDE và công cụ.
    * **Nhược điểm**: Có thể trở nên lặp lại và khó đọc khi cấu hình các thuộc tính có cấu trúc phức tạp (ví dụ: `server.ssl.key-store=...`, `server.ssl.key-store-password=...`).

* **Tệp `.yml` (YAML)**:

    * **Cú pháp**: Dạng cấu trúc cây, phân cấp, sử dụng thụt đầu dòng để biểu thị cấu trúc.
    * **Ưu điểm**: Rất ngắn gọn và dễ đọc đối với các cấu hình có cấu trúc. Loại bỏ sự lặp lại của các tiền tố. Hỗ trợ danh sách và các cấu trúc dữ liệu phức tạp một cách tự nhiên.
    * **Nhược điểm**: Nhạy cảm với việc thụt đầu dòng, có thể dễ gây ra lỗi nếu không cẩn thận.

**Lựa chọn nào?**
Nhiều nhà phát triển ưa thích **YAML (`.yml`)** vì nó giúp tệp cấu hình trở nên ngắn gọn và có tổ chức hơn, đặc biệt là trong các dự án lớn có nhiều cấu hình phức tạp.

#### Ví dụ so sánh

**application.properties:**

```properties
server.port=8080
server.address=127.0.0.1
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
```

**application.yml:**

```yaml
server:
  port: 8080
  address: 127.0.0.1
spring:
  datasource:
    url: jdbc:mysql://localhost/test
    username: dbuser
```

-----

### Câu hỏi 07: Bạn có thể kiểm soát việc ghi log với Spring Boot không? Bằng cách nào?

**Có.** Spring Boot cung cấp khả năng kiểm soát việc ghi log một cách mạnh mẽ và linh hoạt. Bạn có thể kiểm soát nó chủ yếu thông qua tệp `application.properties` hoặc `application.yml`.

**Cách kiểm soát:**

1.  **Thiết lập cấp độ log (Log Levels)**: Bạn có thể thiết lập cấp độ log (TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF) cho toàn bộ ứng dụng hoặc cho các package/lớp cụ thể.

    ```properties
    # Thiết lập cấp độ log cho toàn bộ ứng dụng
    logging.level.root=WARN

    # Thiết lập cấp độ log cho một package cụ thể
    logging.level.com.example.web=DEBUG

    # Thiết lập cấp độ log cho Hibernate
    logging.level.org.hibernate=ERROR
    ```

2.  **Ghi log ra tệp (File Logging)**: Bạn có thể dễ dàng cấu hình để ghi log ra tệp.

    ```properties
    # Ghi log ra một tệp cụ thể
    logging.file.name=myapp.log

    # Hoặc ghi log ra các tệp trong một thư mục cụ thể
    logging.file.path=/var/log/myapp
    ```

3.  **Tùy chỉnh mẫu log (Log Pattern)**: Bạn có thể thay đổi định dạng của các thông điệp log.

    ```properties
    # Tùy chỉnh mẫu log cho console
    logging.pattern.console=%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
    ```

4.  **Sử dụng tệp cấu hình riêng**: Nếu cần các cấu hình phức tạp hơn, bạn có thể cung cấp một tệp cấu hình dành riêng cho hệ thống log mà bạn sử dụng (ví dụ: `logback-spring.xml` cho Logback, `log4j2-spring.xml` cho Log4j2). Spring Boot sẽ tự động phát hiện và sử dụng các tệp này.

-----

### Câu hỏi 08: Theo mặc định, Spring Boot tìm kiếm tệp thuộc tính ở đâu?

Spring Boot sẽ tìm kiếm các tệp `application.properties` và `application.yml` theo một thứ tự ưu tiên nhất định. Các vị trí sau sẽ được quét:

1.  **Một thư mục con `/config` của thư mục hiện tại.** (`./config/`)
2.  **Thư mục hiện tại.** (`./`)
3.  **Một thư mục `/config` trên classpath.** (`classpath:/config/`)
4.  **Thư mục gốc của classpath.** (`classpath:/`)

Danh sách này được sắp xếp theo **thứ tự ưu tiên giảm dần**. Điều này có nghĩa là một thuộc tính trong tệp ở `./config/` sẽ ghi đè lên thuộc tính cùng tên trong tệp ở `classpath:/`. Cách sắp xếp này rất hữu ích, cho phép bạn đóng gói một tệp `application.properties` mặc định bên trong JAR của mình và ghi đè nó bằng một tệp bên ngoài khi triển khai.

-----

### Câu hỏi 09: Làm thế nào để bạn định nghĩa các tệp thuộc tính dành riêng cho profile?

Bạn có thể tạo các tệp thuộc tính dành riêng cho các môi trường khác nhau (ví dụ: dev, test, prod) bằng cách tuân theo quy ước đặt tên: **`application-{profile}.properties`** (hoặc `.yml`).

Ví dụ:

* `application-dev.properties`: Các thuộc tính chỉ dành cho môi trường phát triển.
* `application-prod.properties`: Các thuộc tính chỉ dành cho môi trường sản xuất.
* `application.properties`: Chứa các thuộc tính chung cho tất cả các môi trường.

Khi một profile được kích hoạt (ví dụ: `spring.profiles.active=prod`), Spring Boot sẽ tải cả tệp `application.properties` chung và tệp `application-prod.properties` dành riêng cho profile đó. Các thuộc tính trong tệp dành riêng cho profile sẽ **ghi đè** lên các thuộc tính trong tệp chung.

#### Ví dụ:

**`application.properties` (chung):**

```properties
server.port=8080
app.message=Hello from default
```

**`application-prod.properties` (cho production):**

```properties
server.port=80
app.message=Hello from Production!
```

Nếu bạn chạy ứng dụng với profile `prod`, server sẽ chạy ở cổng 80 và `app.message` sẽ có giá trị là "Hello from Production\!".

-----

### Câu hỏi 10: Làm thế nào để bạn truy cập các thuộc tính được định nghĩa trong các tệp thuộc tính?

Có hai cách chính để truy cập các thuộc tính đã cấu hình:

1.  **Sử dụng annotation `@Value`**: Đây là cách đơn giản để tiêm một giá trị thuộc tính duy nhất vào một trường trong bean của bạn.

    ```java
    @Component
    public class MyComponent {

        @Value("${app.message}")
        private String message;

        public void showMessage() {
            System.out.println(this.message);
        }
    }
    ```

2.  **Sử dụng annotation `@ConfigurationProperties`**: Đây là cách tiếp cận được khuyên dùng khi bạn có nhiều thuộc tính liên quan với nhau. Nó cho phép bạn ánh xạ một nhóm các thuộc tính vào một đối tượng Java (POJO) một cách có cấu trúc và an toàn về kiểu.

#### Ví dụ với `@ConfigurationProperties`

**`application.yml`:**

```yaml
app:
  info:
    name: "My Awesome App"
    version: "1.2.3"
  owner:
    name: "John Doe"
    email: "john.doe@example.com"
```

**Lớp cấu hình:**

```java
@ConfigurationProperties(prefix = "app.info")
public class AppInfoProperties {
    private String name;
    private String version;

    // Getters and Setters
}

// Kích hoạt việc quét @ConfigurationProperties
@EnableConfigurationProperties(AppInfoProperties.class)
@Configuration
public class AppConfig {
}

// Sử dụng trong một service
@Service
public class AppService {
    private final AppInfoProperties appInfo;

    public AppService(AppInfoProperties appInfo) {
        this.appInfo = appInfo;
    }

    public void printAppInfo() {
        System.out.println("App Name: " + appInfo.getName());
        System.out.println("App Version: " + appInfo.getVersion());
    }
}
```

-----

## Câu hỏi 11: Cần định nghĩa những thuộc tính nào để cấu hình MySQL bên ngoài?

Để cấu hình một cơ sở dữ liệu MySQL bên ngoài trong Spring Boot, bạn cần chỉ định URL, tên người dùng (username) và mật khẩu (password) cho `DataSource`.

Các thuộc tính này được định nghĩa trong file `application.properties`:

```properties
# URL kết nối đến cơ sở dữ liệu MySQL của bạn
spring.datasource.url=jdbc:mysql://localhost:3306/ten_database_cua_ban

# Tên người dùng để truy cập database
spring.datasource.username=ten_nguoi_dung

# Mật khẩu để truy cập database
spring.datasource.password=mat_khau
```

Một cách tùy chọn, bạn cũng có thể chỉ định rõ ràng trình điều khiển (driver) JDBC, mặc dù Spring Boot thường có thể tự suy ra điều này từ `url`:

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

Để sử dụng được, bạn cần thêm `mysql-connector-java` vào file `pom.xml` của dự án:

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

Và để tương tác với cơ sở dữ liệu một cách đơn giản, bạn có thể thêm starter `spring-boot-starter-data-jdbc`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

-----

## Câu hỏi 12: Làm thế nào để cấu hình schema mặc định và dữ liệu ban đầu?

Spring Boot có thể tự động khởi tạo schema và nạp dữ liệu vào cơ sở dữ liệu lúc khởi động bằng cách sử dụng các tệp tin SQL.

* **`schema.sql`**: Tệp này chứa các câu lệnh DDL (Ngôn ngữ Định nghĩa Dữ liệu) như `CREATE TABLE` để tạo cấu trúc cơ sở dữ liệu.
* **`data.sql`**: Tệp này chứa các câu lệnh DML (Ngôn ngữ Thao tác Dữ liệu) như `INSERT` để thêm dữ liệu ban đầu.

Các tệp này phải được đặt trong thư mục `src/main/resources`.

**Ví dụ:**

`src/main/resources/schema.sql`:

```sql
DROP TABLE IF EXISTS NGUOI_DUNG;
CREATE TABLE NGUOI_DUNG (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ten_dang_nhap VARCHAR(250) NOT NULL,
    email VARCHAR(250) NOT NULL
);
```

`src/main/resources/data.sql`:

```sql
INSERT INTO NGUOI_DUNG (ten_dang_nhap, email) VALUES
('admin', 'admin@example.com'),
('user', 'user@example.com');
```

Theo mặc định, việc khởi tạo này chỉ được bật cho các cơ sở dữ liệu nhúng (như H2, HSQLDB). Để luôn thực hiện việc này với các cơ sở dữ liệu khác như MySQL, bạn cần thêm thuộc tính sau vào `application.properties`:

```properties
spring.datasource.initialization-mode=always
```

-----

## Câu hỏi 13: "Fat JAR" là gì? Nó khác gì so với JAR gốc?

**Fat JAR** (hay còn gọi là "executable JAR") là một tệp JAR chứa không chỉ mã đã biên dịch của ứng dụng bạn mà còn chứa tất cả các thư viện (dependencies) mà ứng dụng cần để chạy. Điều này làm cho ứng dụng có thể chạy độc lập chỉ với một tệp JAR duy nhất.

Sự khác biệt chính so với JAR gốc (original JAR) là:

* **Phụ thuộc (Dependencies)**: JAR gốc chỉ chứa mã nguồn của bạn. Fat JAR chứa cả mã nguồn của bạn và tất cả các file JAR của thư viện phụ thuộc.
* **Khả năng thực thi**: JAR gốc thường không thể tự thực thi. Fat JAR được tạo bởi Spring Boot có thể chạy trực tiếp từ dòng lệnh bằng `java -jar ten-file.jar` vì nó chứa một trình khởi chạy (launcher) và file `MANIFEST.MF` đã được cấu hình sẵn.

Để tạo một fat JAR, bạn cần sử dụng `spring-boot-maven-plugin` trong file `pom.xml` của mình:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

Sau khi build dự án (ví dụ: `mvn clean package`), bạn sẽ có một tệp fat JAR trong thư mục `target` và có thể chạy nó như sau:

```bash
java -jar ten-ung-dung-cua-ban.jar
```

-----

## Câu hỏi 14: Sự khác biệt giữa container nhúng và WAR là gì?

* **Container nhúng (Embedded Container)**:

    * Là một máy chủ (server) như Tomcat, Jetty được đóng gói ngay bên trong tệp JAR của ứng dụng.
    * Giúp ứng dụng có thể chạy độc lập như một tệp "fat JAR".
    * Mỗi ứng dụng chạy trên máy chủ nhúng của riêng nó. Đây là cách tiếp cận mặc định và được khuyến khích trong Spring Boot để đơn giản hóa việc triển khai và phát triển.

* **WAR (Web Application Archive)**:

    * Là một file định dạng chuẩn để đóng gói một ứng dụng web Java.
    * Nó không thể tự chạy mà phải được triển khai (deploy) lên một máy chủ ứng dụng (Application Server) bên ngoài như Tomcat, WildFly, WebSphere.
    * Một máy chủ ứng dụng có thể chạy nhiều ứng dụng WAR cùng một lúc.

**Ví dụ tạo file JAR với container nhúng (mặc định):**
`pom.xml` chỉ cần starter `web`.

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**Ví dụ để tạo file WAR:**
Trong `pom.xml`, bạn cần thay đổi cách đóng gói thành `war` và đánh dấu `spring-boot-starter-tomcat` là `provided` (vì máy chủ bên ngoài sẽ cung cấp nó).

```xml
<packaging>war</packaging>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

-----

## Câu hỏi 15: Spring Boot hỗ trợ những container nhúng nào?

Spring Boot hỗ trợ ba loại container nhúng chính:

* **Tomcat**: Mặc định. Nó được tự động bao gồm khi bạn thêm `spring-boot-starter-web`.
* **Jetty**
* **Undertow**

Để chuyển sang **Jetty** hoặc **Undertow**, bạn cần loại bỏ `spring-boot-starter-tomcat` mặc định và thêm vào dependency cho container bạn muốn.

**Ví dụ chuyển sang Jetty:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

**Ví dụ chuyển sang Undertow:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

-----

## Câu hỏi 16: Làm thế nào Spring Boot biết cần phải cấu hình những gì?

Spring Boot biết cần cấu hình những gì thông qua cơ chế **Tự động cấu hình (Auto-configuration)**. Quá trình này hoạt động như sau:

1.  **Starter Dependencies**: Khi bạn thêm một "starter" (ví dụ `spring-boot-starter-data-jpa`), bạn đang thêm một tập hợp các thư viện phổ biến vào classpath của ứng dụng.
2.  **Quét `spring.factories`**: Spring Boot sẽ tìm kiếm file `META-INF/spring.factories` trong tất cả các JAR trên classpath. Các lớp tự động cấu hình (Auto-Configuration Classes) được liệt kê trong file này.
3.  **Annotations có điều kiện (`@ConditionalOn...`)**: Mỗi lớp tự động cấu hình sử dụng các annotation điều kiện để quyết định xem có nên được kích hoạt hay không. Ví dụ:
    * `@ConditionalOnClass`: Chỉ kích hoạt nếu một lớp cụ thể có mặt trên classpath.
    * `@ConditionalOnBean`: Chỉ kích hoạt nếu một bean cụ thể đã tồn tại trong context.
    * `@ConditionalOnProperty`: Chỉ kích hoạt nếu một thuộc tính cụ thể trong `application.properties` được đặt.

Khi các điều kiện được đáp ứng, lớp `@Configuration` đó sẽ được nạp và các bean mà nó định nghĩa sẽ được tạo ra, giúp tích hợp ứng dụng của bạn với công nghệ tương ứng.

**Ví dụ về một lớp Auto-Configuration đơn giản:**

```java
// Lớp cấu hình này chỉ được kích hoạt nếu...
@Configuration
// ...có một lớp tên là "DataSource" trong classpath VÀ...
@ConditionalOnClass(DataSource.class)
// ...thuộc tính "myapp.custom.enabled" có giá trị là "true"
@ConditionalOnProperty(name = "myapp.custom.enabled", havingValue = "true")
public class MyCustomAutoConfiguration {

    // Nếu các điều kiện trên đúng, bean này sẽ được tạo
    @Bean
    public MyCustomService myCustomService() {
        return new MyCustomService();
    }
}
```

-----

## Câu hỏi 17: `@EnableAutoConfiguration` làm gì?

Annotation `@EnableAutoConfiguration` kích hoạt cơ chế tự động cấu hình của Spring Boot.

Khi được sử dụng, nó yêu cầu Spring Boot "đoán" xem bạn muốn cấu hình ứng dụng của mình như thế nào dựa trên các JAR phụ thuộc (dependencies) mà bạn đã thêm. Ví dụ, nếu `spring-boot-starter-web` có trên classpath, Spring Boot sẽ giả định bạn đang xây dựng một ứng dụng web và tự động cấu hình DispatcherServlet và một máy chủ Tomcat nhúng.

Annotation này thường không được sử dụng trực tiếp vì nó đã được bao gồm trong annotation tiện ích `@SpringBootApplication`.

-----

## Câu hỏi 18: `@SpringBootApplication` làm gì?

`@SpringBootApplication` là một annotation tiện ích, kết hợp ba annotation quan trọng khác lại với nhau:

1.  **`@Configuration`**: Đánh dấu lớp là một nguồn định nghĩa bean cho application context.
2.  **`@EnableAutoConfiguration`**: Bật cơ chế tự động cấu hình của Spring Boot.
3.  **`@ComponentScan`**: Bật cơ chế quét component (component scanning). Nó yêu cầu Spring quét các component, configuration và service khác trong cùng package với lớp chứa annotation này và các package con.

Việc sử dụng `@SpringBootApplication` tương đương với việc sử dụng cả ba annotation trên.

**Ví dụ về lớp ứng dụng chính:**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // Tương đương @Configuration + @EnableAutoConfiguration + @ComponentScan
public class MyWebAppApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyWebAppApplication.class, args);
    }

}
```

-----

## Câu hỏi 19: Spring Boot có thực hiện component scanning không? Nó tìm kiếm ở đâu theo mặc định?

Có, Spring Boot thực hiện **component scanning** (quét thành phần).

Theo mặc định, việc quét này được kích hoạt bởi annotation `@ComponentScan` (mà `@SpringBootApplication` đã bao gồm). Spring Boot sẽ tự động quét các lớp được chú thích bằng `@Component` (và các stereotype của nó như `@Service`, `@Repository`, `@Controller`) bắt đầu từ package của lớp chính (lớp được chú thích bằng `@SpringBootApplication`) và tất cả các package con của nó.

**Ví dụ cấu trúc thư mục:**

```
com.example.myapp
├── MyWebAppApplication.java   // Lớp chính với @SpringBootApplication
│
├── controller
│   └── MyController.java      // Sẽ được tìm thấy
│
├── service
│   └── MyService.java         // Sẽ được tìm thấy
│
└── model
    └── User.java              // Không phải component, sẽ bị bỏ qua
```

Bạn có thể tùy chỉnh phạm vi quét bằng cách sử dụng thuộc tính `scanBasePackages` trong annotation:

```java
@SpringBootApplication(scanBasePackages = {"com.example.myapp", "com.another.package"})
public class MyWebAppApplication {
    // ...
}
```

-----

## Câu hỏi 20: `DataSource` và `JdbcTemplate` được tự động cấu hình như thế nào?

`DataSource` và `JdbcTemplate` được cấu hình tự động bởi các lớp Auto-Configuration trong module `spring-boot-autoconfigure`.

1.  **`DataSourceAutoConfiguration`**:

    * Lớp này chịu trách nhiệm tạo ra một bean `DataSource`.
    * Nó sẽ được kích hoạt nếu phát hiện có các lớp liên quan đến cơ sở dữ liệu trên classpath (ví dụ: `DataSource`, `EmbeddedDatabaseType`) và bạn đã cung cấp các thuộc tính kết nối cần thiết trong `application.properties`.
    * Các thuộc tính có tiền tố `spring.datasource.*` sẽ được sử dụng để cấu hình `DataSource` (ví dụ: url, username, password).

2.  **`JdbcTemplateAutoConfiguration`**:

    * Lớp này chịu trách nhiệm tạo một bean `JdbcTemplate`.
    * Nó có một điều kiện là `@ConditionalOnBean(DataSource.class)`, nghĩa là nó chỉ được kích hoạt *sau khi* một bean `DataSource` đã được tạo thành công.
    * Khi được tạo, nó sẽ tự động sử dụng `DataSource` có sẵn để khởi tạo `JdbcTemplate`.

Sau khi Spring Boot đã tự động cấu hình xong, bạn có thể tiêm (inject) `DataSource` hoặc `JdbcTemplate` trực tiếp vào các bean của mình bằng `@Autowired`.

**Ví dụ về `application.properties`:**

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/my_db
spring.datasource.username=root
spring.datasource.password=secret
```

**Ví dụ sử dụng `JdbcTemplate` trong một service:**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public UserService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public int countUsers() {
        // JdbcTemplate đã sẵn sàng để sử dụng mà không cần cấu hình thủ công
        return jdbcTemplate.queryForObject("SELECT COUNT(*) FROM USERS", Integer.class);
    }
}
```

-----

## Câu hỏi 21: Tệp `spring.factories` dùng để làm gì?

Tệp `spring.factories`, nằm trong đường dẫn `META-INF/spring.factories` trên classpath, được cơ chế Tự động cấu hình (Auto-Configuration) của Spring Boot sử dụng để xác định vị trí của các lớp cấu hình tự động.

Mỗi module cung cấp các lớp cấu hình tự động cần phải có một tệp `META-INF/spring.factories`. Bên trong tệp này, có một khóa là `org.springframework.boot.autoconfigure.EnableAutoConfiguration`, và giá trị của nó là một danh sách các lớp cấu hình tự động (Auto-Configuration Classes) mà module đó cung cấp.

Khi ứng dụng Spring Boot khởi động, nó sẽ quét tất cả các tệp `spring.factories` trên classpath, đọc danh sách các lớp cấu hình dưới khóa này, và sau đó đánh giá xem có nên áp dụng từng cấu hình hay không dựa trên các điều kiện (`@Conditional`) được định nghĩa trong mỗi lớp.

**Ví dụ về nội dung tệp `spring.factories`:**

Tệp này sẽ liệt kê một hoặc nhiều lớp cấu hình tự động.

```properties
# Comma-separated list of auto-configuration classes
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.MyCustomAutoConfiguration,\
com.example.AnotherAutoConfiguration
```

-----

## Câu hỏi 22: Làm thế nào để tùy chỉnh cấu hình tự động của Spring?

Bạn có thể tùy chỉnh hoặc tạo ra cấu hình tự động của riêng mình bằng cách tạo một module "auto-configuration". Quá trình này bao gồm các bước sau:

1.  **Tạo một Lớp Cấu hình (Configuration Class)**: Đây là một lớp Java được chú thích bằng `@Configuration`. Lớp này sẽ chứa các phương thức được chú thích bằng `@Bean` để định nghĩa các bean mà bạn muốn tự động cấu hình.
2.  **Sử dụng Annotation Điều kiện**: Sử dụng các annotation `@ConditionalOn...` (ví dụ: `@ConditionalOnClass`, `@ConditionalOnProperty`) trên lớp cấu hình hoặc trên các phương thức `@Bean` để chỉ định khi nào cấu hình này nên được áp dụng.
3.  **Tạo tệp `spring.factories`**: Trong module của bạn, tạo một tệp tại `src/main/resources/META-INF/spring.factories`.
4.  **Đăng ký Lớp Cấu hình**: Trong tệp `spring.factories`, thêm một mục nhập để đăng ký lớp cấu hình của bạn.

**Ví dụ:**

Giả sử chúng ta muốn tự động cấu hình một bean `MyService` chỉ khi thuộc tính `my.service.enabled` được đặt thành `true`.

**Bước 1 & 2: Tạo Lớp Cấu hình**

```java
// File: MyServiceAutoConfiguration.java
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
// Chỉ kích hoạt cấu hình này nếu thuộc tính 'my.service.enabled' có giá trị 'true'
@ConditionalOnProperty(name = "my.service.enabled", havingValue = "true")
public class MyServiceAutoConfiguration {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

**Bước 3 & 4: Tạo và đăng ký trong `spring.factories`**

```properties
# File: src/main/resources/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfigure.MyServiceAutoConfiguration
```

Bây giờ, bất kỳ dự án nào bao gồm module này và đặt `my.service.enabled=true` trong `application.properties` sẽ tự động có một bean `MyService`.

-----

## Câu hỏi 23: Các ví dụ về annotation `@Conditional` là gì? Chúng được sử dụng như thế nào?

Spring Boot cung cấp nhiều annotation `@Conditional` để kiểm soát việc kích hoạt các lớp cấu hình tự động. Chúng được sử dụng để chỉ định các điều kiện mà theo đó một cấu hình (`@Configuration`) hoặc một bean (`@Bean`) nên được tạo.

Dưới đây là một số ví dụ phổ biến:

* **`@ConditionalOnBean`**: Chỉ áp dụng cấu hình nếu một bean cụ thể đã tồn tại trong context.
* **`@ConditionalOnMissingBean`**: Chỉ áp dụng cấu hình nếu một bean cụ thể chưa tồn tại.
* **`@ConditionalOnClass`**: Chỉ áp dụng cấu hình nếu một lớp cụ thể có mặt trên classpath.
* **`@ConditionalOnMissingClass`**: Chỉ áp dụng cấu hình nếu một lớp cụ thể không có mặt trên classpath.
* **`@ConditionalOnProperty`**: Chỉ áp dụng cấu hình nếu một thuộc tính (property) trong môi trường Spring có giá trị cụ thể.
* **`@ConditionalOnWebApplication`**: Chỉ áp dụng cấu hình nếu đây là một ứng dụng web.
* **`@ConditionalOnNotWebApplication`**: Chỉ áp dụng cấu hình nếu đây không phải là một ứng dụng web.

**Cách sử dụng:**

Các annotation này được đặt ngay trên một lớp `@Configuration` hoặc một phương thức `@Bean`.

**Ví dụ sử dụng `@ConditionalOnProperty`:**

```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyStorageConfiguration {

    // Bean này chỉ được tạo nếu thuộc tính 'storage.type' được đặt thành 'local'
    @Bean
    @ConditionalOnProperty(name = "storage.type", havingValue = "local")
    public FileStorage localFileStorage() {
        return new LocalFileStorage();
    }

    // Bean này chỉ được tạo nếu thuộc tính 'storage.type' được đặt thành 'cloud'
    @Bean
    @ConditionalOnProperty(name = "storage.type", havingValue = "cloud")
    public FileStorage cloudFileStorage() {
        return new CloudFileStorage();
    }
}
```

-----

## Câu hỏi 24: Spring Boot Actuator cung cấp giá trị gì?

Spring Boot Actuator cung cấp các tính năng sẵn sàng cho môi trường sản xuất (production-ready) để **giám sát** và **quản lý** ứng dụng của bạn.

Giá trị chính mà Actuator mang lại là nó cung cấp các "endpoint" (điểm cuối) mà không cần bạn phải tự viết mã. Các tính năng chính bao gồm:

* **Kiểm tra sức khỏe (Health checks)**: Endpoint `/actuator/health` cho biết trạng thái của ứng dụng (ví dụ: có kết nối được cơ sở dữ liệu, dung lượng ổ đĩa có đủ không).
* **Số liệu (Metrics)**: Endpoint `/actuator/metrics` cung cấp các số liệu chi tiết về hiệu suất như việc sử dụng CPU, bộ nhớ JVM, số lượng request HTTP, v.v.
* **Thông tin ứng dụng (Application Info)**: Endpoint `/actuator/info` hiển thị thông tin tùy chỉnh về ứng dụng như phiên bản, mô tả, thông tin git.
* **Quản lý môi trường (Environment Management)**: Endpoint `/actuator/env` cho phép xem tất cả các thuộc tính cấu hình.

Để sử dụng, bạn chỉ cần thêm dependency `spring-boot-starter-actuator` vào dự án.

**Ví dụ `pom.xml`:**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

Sau khi thêm dependency này, bạn có thể truy cập các endpoint mặc định như `/actuator/health` và `/actuator/info`.

-----

## Câu hỏi 25: Hai giao thức bạn có thể sử dụng để truy cập các endpoint của Actuator là gì?

Bạn có thể sử dụng hai giao thức chính để truy cập các endpoint của Spring Boot Actuator:

1.  **HTTP**: Đây là giao thức phổ biến nhất. Các endpoint được hiển thị dưới dạng các API REST, có thể truy cập qua trình duyệt hoặc bất kỳ công cụ HTTP client nào (như `curl`, Postman).

    * **Ưu điểm**: Dễ sử dụng, có thể truy cập từ bất kỳ đâu, tích hợp tốt với các công cụ giám sát web.
    * **Ví dụ**: `http://localhost:8080/actuator/health`, `http://localhost:8080/actuator/metrics`.

2.  **JMX (Java Management Extensions)**: Là một công nghệ tiêu chuẩn của Java để giám sát và quản lý các ứng dụng. Các endpoint của Actuator cũng có thể được hiển thị qua JMX.

    * **Ưu điểm**: Cung cấp quyền truy cập phong phú hơn vào các hoạt động bên trong của ứng dụng, thường được sử dụng trong các môi trường doanh nghiệp có hệ thống giám sát dựa trên JMX.
    * **Cách truy cập**: Sử dụng các công cụ như JConsole hoặc VisualVM để kết nối đến ứng dụng Spring Boot đang chạy và xem các MBeans (Managed Beans) do Actuator cung cấp.

Bạn có thể cấu hình để bật/tắt các giao thức này và hiển thị/ẩn các endpoint thông qua các thuộc tính trong `application.properties` hoặc `application.yml`, ví dụ:

```properties
management.endpoints.web.exposure.include=* # Hiển thị tất cả các endpoint qua HTTP
management.endpoints.jmx.exposure.include=health,info # Chỉ hiển thị health và info qua JMX
```

Dưới đây là phần tiếp theo của tài liệu hướng dẫn về Spring Boot, với việc loại bỏ các ký tự không cần thiết và tập trung vào nội dung chính:

-----

## Câu hỏi 26: Các endpoint nào của Actuator được cung cấp sẵn?

Spring Boot Actuator cung cấp một loạt các endpoint tích hợp sẵn để giám sát và tương tác với ứng dụng. Dưới đây là một số endpoint quan trọng:

| Endpoint      | Mô tả                                          | Mặc định hiển thị qua Web |
| :------------ | :--------------------------------------------- | :----------------------- |
| `health`      | Hiển thị thông tin sức khỏe của ứng dụng.      | Có                       |
| `info`        | Hiển thị thông tin tùy chỉnh về ứng dụng.      | Có                       |
| `beans`       | Hiển thị danh sách đầy đủ tất cả các Spring bean trong ứng dụng. | Không                    |
| `metrics`     | Hiển thị các thông tin số liệu (metrics) của ứng dụng. | Không                    |
| `loggers`     | Hiển thị và cho phép thay đổi cấp độ ghi log của ứng dụng. | Không                    |
| `env`         | Hiển thị các thuộc tính từ môi trường của Spring. | Không                    |
| `mappings`    | Hiển thị danh sách tất cả các đường dẫn `@RequestMapping`. | Không                    |
| `threaddump`  | Thực hiện một thread dump của JVM.              | Không                    |
| `shutdown`    | Cho phép tắt ứng dụng một cách an toàn (mặc định bị vô hiệu hóa). | Không                    |

Để hiển thị các endpoint khác ngoài `health` và `info` qua web, bạn cần cấu hình trong `application.properties`.

**Ví dụ: Hiển thị các endpoint `metrics` và `loggers` qua web**

```properties
management.endpoints.web.exposure.include=health,info,metrics,loggers
```

**Ví dụ: Hiển thị tất cả các endpoint (không khuyến khích trong môi trường production)**

```properties
management.endpoints.web.exposure.include=*
```

-----

## Câu hỏi 27: Endpoint `info` dùng để làm gì? Làm thế nào để cung cấp dữ liệu cho nó?

Endpoint `info` của Actuator được sử dụng để hiển thị các thông tin tùy ý, không nhạy cảm về ứng dụng của bạn. Đây là một nơi hữu ích để cung cấp siêu dữ liệu (metadata) như tên ứng dụng, phiên bản, mô tả, hoặc thông tin về bản build.

Có hai cách chính để cung cấp dữ liệu cho endpoint `info`:

**1. Sử dụng tệp `application.properties`**

Bạn có thể thêm các thuộc tính với tiền tố `info.`. Spring Boot sẽ tự động thu thập chúng.

**Ví dụ `application.properties`:**

```properties
info.app.name=My Awesome App
info.app.description=Đây là một ứng dụng Spring Boot tuyệt vời.
info.app.version=1.0.0
```

Khi bạn truy cập `/actuator/info`, bạn sẽ thấy một đối tượng JSON chứa các thông tin này.

**2. Triển khai một `InfoContributor` bean**

Để có logic phức tạp hơn (ví dụ: lấy thông tin từ hệ thống), bạn có thể tạo một bean triển khai interface `InfoContributor`.

**Ví dụ `InfoContributor`:**

```java
import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class SystemInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        // Thêm thông tin động vào endpoint /info
        builder.withDetail("os_name", System.getProperty("os.name"));
        builder.withDetail("java_version", System.getProperty("java.version"));
    }
}
```

-----

## Câu hỏi 28: Làm thế nào để thay đổi cấp độ ghi log của một package bằng endpoint `loggers`?

Endpoint `loggers` của Actuator cho phép bạn xem và thay đổi cấp độ ghi log (logging level) của ứng dụng tại thời điểm chạy mà không cần khởi động lại.

**Bước 1: Hiển thị endpoint `loggers`**

Trước tiên, bạn cần hiển thị endpoint này qua web bằng cách thêm vào `application.properties`:

```properties
management.endpoints.web.exposure.include=health,info,loggers
```

**Bước 2: Xem cấp độ log hiện tại**

Bạn có thể xem tất cả các logger bằng cách truy cập `/actuator/loggers`, hoặc xem một logger cụ thể bằng cách truy cập `/actuator/loggers/{tên_package_hoặc_lớp}`.

**Ví dụ xem logger cho package `com.example.demo`:**

```bash
curl http://localhost:8080/actuator/loggers/com.example.demo
```

Kết quả có thể trông như thế này:

```json
{
  "configuredLevel": null,
  "effectiveLevel": "INFO"
}
```

**Bước 3: Thay đổi cấp độ log**

Để thay đổi cấp độ log, bạn gửi một yêu cầu `POST` đến URL của logger đó với body là một JSON chỉ định `configuredLevel` mới. Các cấp độ hợp lệ là `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`, `OFF`, hoặc `null` (để reset về mặc định).

**Ví dụ: Thay đổi cấp độ log của package `com.example.demo` thành `DEBUG`:**

```bash
curl -i -X POST \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}' \
  http://localhost:8080/actuator/loggers/com.example.demo
```

Sau khi thực hiện lệnh này, tất cả các log ở cấp độ `DEBUG` trở lên từ package `com.example.demo` sẽ được hiển thị.

-----

## Câu hỏi 29: Làm thế nào để truy cập một endpoint bằng cách sử dụng tag?

Việc sử dụng tag cho phép bạn lọc kết quả từ các endpoint của Actuator, đặc biệt hữu ích với endpoint `metrics`. Bạn có thể truy vấn các số liệu cho các chiều (dimension) cụ thể.

Cú pháp để sử dụng tag là thêm tham số `tag=KEY:VALUE` vào URL.

**Ví dụ 1: Lọc các request HTTP có mã trạng thái là 200**
Endpoint `http.server.requests` theo dõi các request HTTP. Nó có các tag như `status`, `method`, `uri`.

```bash
# Lấy số liệu chỉ cho các request có status code là 200
curl http://localhost:8080/actuator/metrics/http.server.requests?tag=status:200
```

**Ví dụ 2: Lọc bộ nhớ heap đã sử dụng**
Endpoint `jvm.memory.used` có tag là `area` (`heap` hoặc `nonheap`).

```bash
# Lấy số liệu bộ nhớ đã sử dụng chỉ cho vùng heap
curl http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:heap
```

Bạn cũng có thể kết hợp nhiều tag bằng cách lặp lại tham số `tag`.

**Ví dụ 3: Lọc các request GET có mã trạng thái 200**

```bash
# Sử dụng nhiều tag để lọc
curl http://localhost:8080/actuator/metrics/http.server.requests?tag=method:GET&tag=status:200
```

-----

## Câu hỏi 30: `metrics` dùng để làm gì?

Endpoint `metrics` của Actuator được sử dụng để **kiểm tra các số liệu** (metrics) được ứng dụng thu thập trong quá trình chạy. Nó cung cấp một cái nhìn sâu sắc về hiệu suất và hành vi của ứng dụng.

**Các tính năng chính:**

* Cung cấp danh sách tất cả các metrics có sẵn.
* Cho phép xem chi tiết một metric cụ thể, ví dụ: `/actuator/metrics/process.cpu.usage`.
* Cho phép lọc dữ liệu bằng cách sử dụng các tag, ví dụ: `/actuator/metrics/jvm.memory.used?tag=area:heap`.

**Một số metrics có sẵn:**

* **Metrics JVM**: Sử dụng bộ nhớ, thống kê bộ gom rác (garbage collector), thông tin về thread.
* **Metrics hệ thống**: Sử dụng CPU, số lượng core.
* **Metrics ứng dụng**: Số lượng request HTTP, thời gian phản hồi.
* **Metrics nguồn dữ liệu (DataSource)**: Số lượng kết nối đang hoạt động, đang chờ.

Theo mặc định, endpoint này không được hiển thị qua web. Bạn cần kích hoạt nó trong `application.properties`.

**Ví dụ: Hiển thị endpoint `metrics`:**

```properties
management.endpoints.web.exposure.include=metrics
```

Sau khi kích hoạt, bạn có thể truy cập `/actuator/metrics` để xem danh sách các metrics có sẵn và đi sâu vào từng metric để phân tích.

-----

## Câu hỏi 31: Làm thế nào để tạo một metric tùy chỉnh (custom metric) có hoặc không có tag?

Bạn có thể tạo các metric tùy chỉnh trong Spring Boot Actuator bằng cách sử dụng **`MeterRegistry`** từ Micrometer, một facade (lớp vỏ) cho các số liệu ứng dụng. Micrometer cho phép bạn đăng ký nhiều loại "meter" khác nhau sẽ được hiển thị qua endpoint `/actuator/metrics`.

Các loại meter cơ bản bao gồm:

* Counter
* Gauge
* Timer
* Distribution Summary

Để tạo một metric, bạn có thể đăng ký nó trực tiếp với `MeterRegistry`. Bạn có thể thêm các **"tag"** (thẻ) để tạo ra các chiều (dimensions) cho metric của mình, giúp cho việc lọc và phân tích dữ liệu trở nên linh hoạt hơn.

**Ví dụ tạo metric với `MeterRegistry`:**
Giả sử bạn có một service và muốn đếm số lần một đối tượng được lưu trữ.

```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class ObjectStorageService {

    private final Counter objectCounterWithTags;
    private final Counter objectCounterWithoutTags;

    public ObjectStorageService(MeterRegistry meterRegistry) {
        // 1. Tạo một counter CÓ TAGS để phân biệt theo loại lưu trữ
        this.objectCounterWithTags = meterRegistry.counter("storage.object.count", "type", "database");

        // 2. Tạo một counter KHÔNG CÓ TAGS
        this.objectCounterWithoutTags = meterRegistry.counter("storage.simple.count");
    }

    public void saveObject() {
        // Tăng giá trị của counter mỗi khi phương thức được gọi
        this.objectCounterWithTags.increment();
        this.objectCounterWithoutTags.increment();
        System.out.println("Đối tượng đã được lưu và metric đã được tăng.");
    }
}
```

Sau khi gọi phương thức `saveObject()`, bạn có thể truy cập `/actuator/metrics/storage.object.count` để xem metric của mình.

-----

## Câu hỏi 32: Health Indicator là gì?

**Health Indicator** (Chỉ báo Sức khỏe) là một thành phần được endpoint `/actuator/health` sử dụng để kiểm tra xem một phần của hệ thống có đang ở trạng thái tốt và sẵn sàng xử lý các yêu cầu hay không.

Endpoint `/actuator/health` sẽ tổng hợp kết quả từ tất cả các `HealthIndicator` đã được đăng ký để đưa ra trạng thái sức khỏe cuối cùng cho toàn bộ ứng dụng. Các công cụ giám sát thường sử dụng endpoint này để tự động kiểm tra trạng thái hệ thống và gửi cảnh báo nếu có sự cố. Nó cũng rất hữu ích trong các kiến trúc có tính sẵn sàng cao (High Availability), nơi mà các bộ cân bằng tải (Load Balancer) có thể dùng nó để quyết định xem một instance có đủ khỏe để nhận traffic hay không.

Để tạo một `HealthIndicator` tùy chỉnh, bạn cần tạo một Spring Bean triển khai (implements) interface `HealthIndicator`.

**Ví dụ tạo một `HealthIndicator` tùy chỉnh:**

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyCustomHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean isSystemReady = checkSomething(); // Logic kiểm tra của bạn
        if (!isSystemReady) {
            // Nếu hệ thống không sẵn sàng, trả về trạng thái DOWN kèm chi tiết
            return Health.down().withDetail("reason", "Dịch vụ phụ thuộc không phản hồi").build();
        }
        
        // Nếu mọi thứ ổn, trả về trạng thái UP
        return Health.up().withDetail("system-ready", true).build();
    }

    private boolean checkSomething() {
        // Thực hiện kiểm tra ở đây, ví dụ: gọi một dịch vụ bên ngoài
        return true; // Giả sử là ổn
    }
}
```

-----

## Câu hỏi 33: Các Health Indicator nào được cung cấp sẵn?

Spring Boot Actuator tự động cấu hình nhiều `HealthIndicator` có sẵn khi nó phát hiện các dependency tương ứng trên classpath. Điều này có nghĩa là bạn không cần phải viết mã kiểm tra cho các thành phần phổ biến.

Dưới đây là một số chỉ báo sức khỏe được cung cấp sẵn:

* **`DiskSpaceHealthIndicator`**: Kiểm tra dung lượng đĩa còn trống.
* **`DataSourceHealthIndicator`**: Kiểm tra trạng thái của một `DataSource` (kết nối cơ sở dữ liệu).
* **`RedisHealthIndicator`**: Kiểm tra xem máy chủ Redis có đang hoạt động không.
* **`RabbitHealthIndicator`**: Kiểm tra xem máy chủ RabbitMQ có đang hoạt động không.
* **`MongoHealthIndicator`**: Kiểm tra xem cơ sở dữ liệu Mongo có đang hoạt động không.
* **`CassandraHealthIndicator`**: Kiểm tra xem cơ sở dữ liệu Cassandra có đang hoạt động không.
* **`ElasticsearchHealthIndicator`**: Kiểm tra xem cụm Elasticsearch có đang hoạt động không.
* **`JmsHealthIndicator`**: Kiểm tra xem một JMS broker có đang hoạt động không.
* **`MailHealthIndicator`**: Kiểm tra xem máy chủ mail có đang hoạt động không.

Ngoài ra, Actuator cũng cung cấp các phiên bản "Reactive" của các chỉ báo này cho các ứng dụng sử dụng Spring WebFlux.

-----

## Câu hỏi 34: Trạng thái của Health Indicator là gì?

Trạng thái của `Health Indicator` được dùng để thông báo cho Spring Actuator biết liệu thành phần hệ thống mà nó đang kiểm tra có hoạt động chính xác hay không.

Mỗi `HealthIndicator` phải trả về một trong các trạng thái sau:

* **`UP`**: Thành phần đang hoạt động như mong đợi.
* **`DOWN`**: Thành phần đã gặp một lỗi không mong muốn.
* **`OUT_OF_SERVICE`**: Thành phần đã được đưa ra khỏi dịch vụ và không nên được sử dụng.
* **`UNKNOWN`**: Trạng thái của thành phần không xác định.
* Trạng thái tùy chỉnh do người dùng định nghĩa.

Spring Actuator sử dụng một `HealthAggregator` để tổng hợp các trạng thái từ tất cả các `HealthIndicator` và quyết định trạng thái cuối cùng cho toàn bộ ứng dụng. Theo mặc định, `OrderedHealthAggregator` sẽ sắp xếp các trạng thái theo thứ tự ưu tiên (`DOWN`, `OUT_OF_SERVICE`, `UP`, `UNKNOWN`) và lấy trạng thái có độ ưu tiên cao nhất làm trạng thái cuối cùng.

-----

## Câu hỏi 35: Các trạng thái Health Indicator nào được cung cấp sẵn?

Spring Actuator cung cấp sẵn các trạng thái sau đây:

* **`UP`**: Thành phần hoặc hệ thống con đang hoạt động như mong đợi.
* **`DOWN`**: Thành phần hoặc hệ thống con đã gặp một lỗi không mong muốn.
* **`OUT_OF_SERVICE`**: Thành phần hoặc hệ thống con đã được đưa ra khỏi dịch vụ và không nên được sử dụng.
* **`UNKNOWN`**: Trạng thái của thành phần hoặc hệ thống con không xác định.

Dựa trên các trạng thái này, Spring sẽ tự động ánh xạ chúng tới các mã trạng thái HTTP khi bạn truy cập endpoint `/actuator/health`.

* `UP` -\> HTTP **200**
* `UNKNOWN` -\> HTTP **200**
* `DOWN` -\> HTTP **503** (Service Unavailable)
* `OUT_OF_SERVICE` -\> HTTP **503** (Service Unavailable)

Bạn có thể tùy chỉnh ánh xạ này bằng cách sử dụng thuộc tính `management.health.status.http-mapping` trong file `application.properties`.

**Ví dụ: Thay đổi mã HTTP cho trạng thái `DOWN` thành `500`**

```properties
management.health.status.http-mapping.DOWN=500
```

-----

## Câu hỏi 36: Làm thế nào để thay đổi thứ tự ưu tiên mức độ nghiêm trọng của trạng thái Health Indicator?

Spring Actuator cho phép bạn thay đổi thứ tự ưu tiên mức độ nghiêm trọng của các trạng thái bằng cách sử dụng thuộc tính `management.health.status.order`.

Thứ tự mặc định là: `DOWN`, `OUT_OF_SERVICE`, `UP`, `UNKNOWN`.

`OrderedHealthAggregator` sử dụng thứ tự này để quyết định trạng thái tổng hợp cuối cùng của ứng dụng. Nó sẽ lấy trạng thái có độ ưu tiên cao nhất (xuất hiện đầu tiên trong danh sách) từ tất cả các chỉ báo.

Bạn có thể thay đổi thứ tự này hoặc thêm các trạng thái tùy chỉnh của riêng mình.

**Ví dụ: Thêm một trạng thái tùy chỉnh `FATAL` với độ ưu tiên cao nhất:**

```properties
management.health.status.order=FATAL, DOWN, OUT_OF_SERVICE, UP, UNKNOWN
```

Với cấu hình này, nếu bất kỳ `HealthIndicator` nào trả về trạng thái `FATAL`, trạng thái tổng hợp của toàn bộ ứng dụng sẽ là `FATAL`.

-----

## Câu hỏi 37: Tại sao bạn muốn tận dụng hệ thống giám sát bên ngoài của bên thứ ba?

Việc sử dụng một hệ thống giám sát bên ngoài (như Prometheus, Datadog, Dynatrace) mang lại nhiều lợi ích vì bạn có thể sử dụng các chức năng giám sát mạnh mẽ mà không cần phải tự mình xây dựng chúng.

Các hệ thống này thường cung cấp:

* **Lưu trữ bền vững (Durable storage)**: Lưu trữ dữ liệu metrics trong thời gian dài.
* **Thu thập dữ liệu hiệu quả**: Khả năng xử lý lượng lớn dữ liệu từ nhiều nguồn.
* **Truy vấn dữ liệu**: Cung cấp ngôn ngữ truy vấn mạnh mẽ để phân tích dữ liệu.
* **Trực quan hóa dữ liệu (Data visualization)**: Tạo các biểu đồ và đồ thị đẹp mắt.
* **Bảng điều khiển (Dashboards)** có thể cấu hình.
* **Hệ thống cảnh báo (Alerting)** có thể cấu hình.

Spring Actuator tích hợp liền mạch với các hệ thống này thông qua **Micrometer**. Để tích hợp, bạn chỉ cần thêm dependency của registry tương ứng vào dự án của mình.

**Ví dụ: Tích hợp với Prometheus**

Thêm dependency sau vào `pom.xml`:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Spring Boot sẽ tự động cấu hình một endpoint `/actuator/prometheus` mà máy chủ Prometheus có thể thu thập (scrape) dữ liệu từ đó.

-----

## Câu hỏi 38: Khi nào bạn muốn sử dụng annotation `@SpringBootTest`?

Bạn nên sử dụng annotation `@SpringBootTest` bất cứ khi nào bạn viết một **bài kiểm thử tích hợp (Integration Test)** cho một ứng dụng Spring Boot.

Không giống như unit test chỉ kiểm thử một lớp đơn lẻ, integration test kiểm thử sự phối hợp giữa nhiều thành phần với nhau. `@SpringBootTest` giúp việc này trở nên dễ dàng bằng cách khởi động một `ApplicationContext` hoàn chỉnh (hoặc một phần của nó) cho bài kiểm thử của bạn.

Các tính năng hữu ích mà nó cung cấp bao gồm:

* Tự động tạo `ApplicationContext`.
* Cung cấp một môi trường web cho việc kiểm thử (mock hoặc nhúng).
* Khả năng tiêm các mock bean bằng `@MockBean` và `@SpyBean`.
* Tự động cấu hình cho việc kiểm thử MVC, JPA, JSON, và nhiều hơn nữa.

**Ví dụ một bài kiểm thử tích hợp sử dụng `@SpringBootTest`:**

```java
import static org.assertj.core.api.Assertions.assertThat;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

// Annotation này sẽ khởi động toàn bộ ứng dụng Spring
@SpringBootTest
public class MyApplicationIntegrationTest {

    @Autowired
    private MyService myService; // Bạn có thể tiêm các bean từ context

    @Test
    public void contextLoads() {
        // Kiểm tra xem service đã được tiêm thành công hay chưa
        assertThat(myService).isNotNull();
    }
}
```

-----

## Câu hỏi 39: `@SpringBootTest` tự động cấu hình những gì?

Annotation `@SpringBootTest` tự động cấu hình môi trường cần thiết để chạy các bài kiểm thử tích hợp. Cụ thể, nó sẽ:

1.  **Tự động cấu hình `ApplicationContext`**: Nó sẽ tìm kiếm lớp được chú thích bằng `@SpringBootApplication` hoặc `@SpringBootConfiguration` để tải các định nghĩa bean và tạo ra một context cho bài kiểm thử.
2.  **Tự động cấu hình các công cụ kiểm thử**: Nó cung cấp các tiện ích và cấu hình tự động cho việc kiểm thử.

Bạn có thể kiểm thử một "lát cắt" (slice) của ứng dụng thay vì toàn bộ context bằng cách sử dụng các annotation chuyên dụng hơn. Các annotation này được xây dựng trên nền tảng của `@SpringBootTest` và `@AutoConfigure...` để đơn giản hóa việc viết test cho các tầng cụ thể:

* **`@WebMvcTest`**: Chỉ kiểm thử tầng controller của MVC, không tải các tầng service hay repository.
* **`@DataJpaTest`**: Chỉ kiểm thử tầng repository của JPA.
* **`@JsonTest`**: Chỉ kiểm thử việc serialize và deserialize JSON.
* **`@RestClientTest`**: Chỉ kiểm thử các `RestClient`.

**Ví dụ sử dụng `@WebMvcTest` để kiểm thử một controller:**

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;

// Chỉ tải các bean liên quan đến web và controller được chỉ định
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mockMvc; // MockMvc được tự động cấu hình

    @Test
    public void shouldReturnOk() throws Exception {
        mockMvc.perform(get("/hello"))
               .andExpect(status().isOk());
    }
}
```

-----

## Câu hỏi 40: `spring-boot-starter-test` mang lại những dependency nào?

`spring-boot-starter-test` là một starter tiện ích, nó gộp nhiều thư viện kiểm thử phổ biến nhất vào dự án của bạn với chỉ một dependency duy nhất.

Các dependency chính bao gồm:

* **JUnit 5**: Framework kiểm thử đơn vị tiêu chuẩn cho Java.
* **Spring Test & Spring Boot Test**: Hỗ trợ tích hợp cho việc kiểm thử các ứng dụng Spring và Spring Boot.
* **AssertJ**: Một thư viện assertion linh hoạt (fluent assertion).
* **Hamcrest**: Một thư viện cho các đối tượng "matchers".
* **Mockito**: Một framework phổ biến để tạo các đối tượng giả (mock).
* **JSONassert**: Một thư viện assertion cho JSON.
* **JsonPath**: Cung cấp XPath cho JSON.

Việc có sẵn các thư viện này giúp bạn có thể bắt đầu viết các bài kiểm thử đa dạng (unit, integration, mock) một cách nhanh chóng mà không cần phải quản lý nhiều dependency riêng lẻ.

-----

## Câu hỏi 41: Làm thế nào để thực hiện kiểm thử tích hợp (integration testing) với `@SpringBootTest` cho một ứng dụng web?

Kiểm thử tích hợp (integration test) là kiểm tra sự tương tác giữa nhiều thành phần của hệ thống để đảm bảo chúng hoạt động cùng nhau như mong đợi. Khi kiểm thử các thành phần web (như Controller), bài kiểm thử nên thực hiện một yêu cầu HTTP thực sự và kiểm tra phản hồi HTTP nhận được.

Spring Boot cung cấp hai cách chính để thực hiện kiểm thử tích hợp cho các thành phần web:

### 1\. Sử dụng `MockMvc`

Cách tiếp cận này mô phỏng (mock) môi trường MVC của Spring mà không cần khởi động một máy chủ web thực sự. Nó nhanh hơn và lý tưởng để kiểm tra logic của controller và ánh xạ yêu cầu (request mapping).

**Ví dụ:**

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc // Tự động cấu hình MockMvc
public class CityControllerWebMockMvcTest {

    @Autowired
    private MockMvc mvc; // Tiêm MockMvc để sử dụng

    @Test
    public void shouldReturnCityList() throws Exception {
        mvc.perform(get("/api/cities"))
           .andExpect(status().isOk())
           .andExpect(content().string("[\"Hanoi\",\"Ho Chi Minh City\"]"));
    }
}
```

-----

### 2\. Sử dụng Container nhúng (Embedded Container)

Cách tiếp cận này khởi động một máy chủ web thực sự trên một cổng ngẫu nhiên. Nó cho phép bạn thực hiện các yêu cầu HTTP đến ứng dụng của mình bằng một HTTP client như `TestRestTemplate`. Đây là một bài kiểm thử tích hợp toàn diện hơn.

**Ví dụ:**

```java
import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.test.web.server.LocalServerPort;

// Khởi động server trên một cổng ngẫu nhiên
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class CityControllerWebIntegrationTest {

    @LocalServerPort
    private int port; // Lấy cổng mà server đang chạy

    @Autowired
    private TestRestTemplate restTemplate; // Client để gửi yêu cầu HTTP

    @Test
    public void shouldReturnCityList() {
        String url = "http://localhost:" + port + "/api/cities";
        String response = this.restTemplate.getForObject(url, String.class);
        
        assertThat(response).contains("Hanoi", "Ho Chi Minh City");
    }
}
```

-----

## Câu hỏi 42: Khi nào bạn muốn sử dụng `@WebMvcTest`? Nó tự động cấu hình những gì?

Bạn nên sử dụng **`@WebMvcTest`** khi muốn viết các bài kiểm thử chỉ tập trung vào **tầng web (web layer)** của ứng dụng, chẳng hạn như các `Controller`.

`@WebMvcTest` sẽ chỉ tạo một `ApplicationContext` chứa các thành phần liên quan đến web, bỏ qua các thành phần khác như `@Service` hay `@Repository`. Điều này giúp bài kiểm thử nhẹ hơn và nhanh hơn so với việc tải toàn bộ ứng dụng. Nếu controller của bạn phụ thuộc vào các service khác, bạn cần phải **giả lập (mock)** chúng bằng `@MockBean`.

Nó tự động cấu hình các thành phần sau:

* **`MockMvc`**
* Các lớp **`@Controller`**, **`@ControllerAdvice`**, **`@JsonComponent`**
* Các lớp **`@Converter`**, **`@Filter`**, **`@WebMvcConfigurer`**

**Ví dụ:**

```java
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import java.util.Arrays;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

// Chỉ kiểm thử UserController
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean // Giả lập UserService vì nó không được tải trong @WebMvcTest
    private UserService userService;

    @Test
    public void testGetUsers() throws Exception {
        // Định nghĩa hành vi của mock service
        when(userService.getAllUsers()).thenReturn(Arrays.asList(new User("testuser")));

        mockMvc.perform(get("/users"))
               .andExpect(status().isOk());
    }
}
```

-----

## Câu hỏi 43: Sự khác biệt giữa `@MockBean` và `@Mock` là gì?

Sự khác biệt chính nằm ở nguồn gốc và cách chúng tích hợp với Spring.

### `@Mock`

* **Nguồn gốc**: Đến từ framework **Mockito**.
* **Chức năng**: Tạo một đối tượng giả (mock object).
* **Tích hợp**: Nó **không** tự động tích hợp với `ApplicationContext` của Spring. Bạn phải tự tiêm (inject) mock này vào lớp đang được kiểm thử, thường là bằng cách sử dụng `@InjectMocks` của Mockito. Nó chỉ hoạt động trong phạm vi của lớp kiểm thử đó.

**Ví dụ với `@Mock`:**

```java
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

// Sử dụng extension của Mockito để khởi tạo các mock
@ExtendWith(MockitoExtension.class)
class MyServiceUnitTest {

    @Mock // Tạo một mock của UserRepository
    private UserRepository userRepository;

    @InjectMocks // Tiêm mock ở trên vào MyService
    private MyService myService;

    // ... các bài kiểm thử unit test cho MyService ...
}
```

-----

### `@MockBean`

* **Nguồn gốc**: Đến từ **Spring Boot Test** (`spring-boot-test`).
* **Chức năng**: Tạo một mock object của Mockito và **thay thế bean thật** có cùng kiểu trong `ApplicationContext` của Spring bằng mock đó.
* **Tích hợp**: Bất kỳ bean nào khác trong context mà phụ thuộc (`@Autowired`) vào bean bị mock sẽ nhận được đối tượng mock này thay vì bean thật. Nó hữu ích trong các bài kiểm thử tích hợp (`@SpringBootTest`, `@WebMvcTest`).

**Ví dụ với `@MockBean`:**

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.beans.factory.annotation.Autowired;

@SpringBootTest
class MyControllerIntegrationTest {

    @Autowired
    private MyController myController; // Controller thật từ context

    @MockBean // Tạo mock cho MyService và thay thế bean thật trong context
    private MyService myService;

    // ... các bài kiểm thử tích hợp cho MyController, 
    // myController sẽ sử dụng myService đã được mock ...
}
```

Tóm lại, **`@MockBean`** là một công cụ mạnh mẽ của Spring Boot để thay thế bean trong toàn bộ context cho mục đích kiểm thử tích hợp, trong khi **`@Mock`** là một công cụ cơ bản của Mockito để tạo mock cho các bài kiểm thử đơn vị (unit test).

-----

## Câu hỏi 44: Khi nào bạn muốn sử dụng `@DataJpaTest`? Nó tự động cấu hình những gì?

Bạn nên sử dụng **`@DataJpaTest`** khi muốn viết các bài kiểm thử tích hợp chỉ tập trung vào các thành phần liên quan đến **JPA**, chẳng hạn như các `Entity` và `Repository`.

`@DataJpaTest` sẽ không tải toàn bộ ứng dụng mà chỉ tải các thành phần cần thiết cho tầng dữ liệu, giúp bài kiểm thử nhanh và gọn gàng hơn.

Nó tự động cấu hình các thành phần sau:

* Một **cơ sở dữ liệu nhúng trong bộ nhớ** (in-memory embedded database).
* Quét và cấu hình các bean `@Entity`.
* Quét và cấu hình các **Spring Data Repository**.
* Cấu hình một **`TestEntityManager`**, một tiện ích để thao tác với các entity trong các bài kiểm thử.

Quan trọng là, mỗi bài kiểm thử `@DataJpaTest` đều được thực hiện trong một **giao dịch (transaction)** và sẽ được **rollback** sau khi kết thúc, đảm bảo các bài kiểm thử độc lập với nhau.

**Ví dụ:**

```java
import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;

@DataJpaTest // Chỉ tải các thành phần liên quan đến JPA
public class ProductRepositoryTest {

    @Autowired
    private TestEntityManager entityManager; // Dùng để chuẩn bị dữ liệu

    @Autowired
    private ProductRepository productRepository; // Repository cần kiểm thử

    @Test
    public void whenFindByName_thenReturnProduct() {
        // Chuẩn bị dữ liệu
        Product product = new Product("Laptop");
        entityManager.persist(product);
        entityManager.flush();

        // Thực hiện hành động
        Product found = productRepository.findByName(product.getName());

        // Kiểm tra kết quả
        assertThat(found.getName()).isEqualTo(product.getName());
    }
}
```