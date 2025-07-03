### Truy Cập Dữ Liệu và Giao Dịch trong Spring

Phần này sẽ đi sâu vào cách Spring xử lý việc tương tác với cơ sở dữ liệu và quản lý các giao dịch.

### Câu hỏi 01: Sự khác biệt giữa checked và unchecked exceptions là gì? Tại sao Spring ưu tiên unchecked exceptions? Hệ thống phân cấp Data Access Exception là gì?

**Checked Exceptions (Ngoại lệ được kiểm tra):**

* Là các ngoại lệ kế thừa từ `java.lang.Exception` (nhưng không phải `RuntimeException`).
* **Buộc phải xử lý**: Trình biên dịch yêu cầu bạn phải khai báo chúng trong mệnh đề `throws` của phương thức hoặc xử lý chúng trong một khối `try-catch`.
* **Ưu điểm**: Cung cấp phản hồi ngay lúc biên dịch, đảm bảo lập trình viên biết và xử lý các tình huống ngoại lệ.
* **Nhược điểm**: Có thể dẫn đến mã nguồn lộn xộn (cluttered code) và tạo ra sự kết dính chặt chẽ giữa bên gọi và bên được gọi.

**Unchecked Exceptions (Ngoại lệ không được kiểm tra):**

* Là các ngoại lệ kế thừa từ `java.lang.RuntimeException`.
* **Không buộc phải xử lý**: Bạn không bắt buộc phải khai báo hay xử lý chúng.
* **Ưu điểm**: Giảm sự lộn xộn trong mã nguồn và giảm sự kết dính giữa các thành phần. Lập trình viên có quyền tự do lựa chọn có xử lý ngoại lệ hay không.
* **Nhược điểm**: Thiếu phản hồi lúc biên dịch, có thể dẫn đến việc bỏ sót xử lý lỗi.

**Tại sao Spring ưu tiên Unchecked Exceptions?**
Spring ưu tiên sử dụng unchecked exceptions vì chúng không buộc client phải xử lý các ngoại lệ mà họ không thể phục hồi. Việc bắt và bao bọc các checked exceptions (như `SQLException`) trong các khối `try-catch` lặp đi lặp lại được coi là một thực hành không tốt. Bằng cách chuyển đổi các ngoại lệ dành riêng cho công nghệ (như `SQLException`) thành hệ thống phân cấp `DataAccessException` (là một unchecked exception), Spring cho phép lập trình viên chỉ bắt những ngoại lệ mà họ thực sự có thể xử lý, giúp mã nguồn sạch sẽ và dễ bảo trì hơn.

**Hệ thống phân cấp Data Access Exception:**
Đây là một hệ thống phân cấp các `RuntimeException` do Spring cung cấp. Mục đích của nó là tạo ra một lớp trừu tượng lên trên các API truy cập dữ liệu cụ thể (như JDBC, Hibernate, JPA). Điều này giúp:

* **Tách biệt (Decoupling)**: Mã nguồn của bạn không bị phụ thuộc vào các ngoại lệ của một công nghệ cụ thể (ví dụ: `SQLException`).
* **Dễ chuyển đổi**: Dễ dàng chuyển đổi từ công nghệ này sang công nghệ khác (ví dụ: từ JDBC sang JPA) mà không cần thay đổi mã xử lý ngoại lệ.

Một số ví dụ về các ngoại lệ trong hệ thống này: `CannotAcquireLockException`, `DataIntegrityViolationException`, `IncorrectResultSizeDataAccessException`.

### Câu hỏi 02: Bạn cấu hình một DataSource trong Spring như thế nào? Bean nào rất hữu ích cho các cơ sở dữ liệu phát triển/kiểm thử?

**Cách cấu hình DataSource:**
Cách cấu hình một `DataSource` phụ thuộc vào môi trường ứng dụng:

* **Ứng dụng độc lập (Standalone)**: Cấu hình `DataSource` như một bean bên trong một lớp `@Configuration`. Bạn có thể sử dụng các implementation như `DriverManagerDataSource` của Spring hoặc các connection pool phổ biến như HikariCP, DBCP, C3P0.
* **Ứng dụng Spring Boot**: Cách đơn giản nhất là cấu hình trong tệp `application.properties` hoặc `application.yml`. Spring Boot sẽ tự động tạo và cấu hình một `DataSource` (thường là HikariCP) dựa trên các thuộc tính đó.
* **Trong Application Server**: `DataSource` nên được lấy từ JNDI. Application Server sẽ chịu trách nhiệm tạo và quản lý `DataSource`.

**Bean hữu ích cho môi trường Dev/Test:**

* `EmbeddedDatabaseBuilder`: Cho phép bạn dễ dàng cấu hình một cơ sở dữ liệu nhúng (như H2, HSQLDB) bằng cách sử dụng một builder pattern. Nó cũng cho phép chạy các script SQL để khởi tạo schema và dữ liệu. Đây là lựa chọn cực kỳ hữu ích cho các bài test hoặc môi trường phát triển nhanh.
* `DataSourceInitializer` / `ResourceDatabasePopulator`: Cho phép bạn chạy các script khởi tạo schema và dữ liệu trên bất kỳ `DataSource` nào, không chỉ riêng cơ sở dữ liệu nhúng.

#### Ví dụ Code: Cấu hình DataSource cho Dev/Test

```java
@Configuration
public class DataSourceConfig {

    // Sử dụng EmbeddedDatabaseBuilder để tạo một H2 DataSource cho môi trường test
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2) // Chỉ định loại CSDL nhúng
                .addScript("classpath:schema.sql") // Script tạo bảng
                .addScript("classpath:data.sql")   // Script chèn dữ liệu mẫu
                .build();
    }
}
```

### Câu hỏi 03: Mẫu thiết kế Template là gì và JDBC Template là gì?

**Mẫu thiết kế Template (Template Method Pattern):**
Đây là một mẫu thiết kế hành vi (behavioral design pattern). Nó định nghĩa bộ khung của một thuật toán trong một phương thức, và giao lại việc triển khai một số bước cụ thể cho các lớp con. Điều này cho phép các lớp con định nghĩa lại một số bước của thuật toán mà không làm thay đổi cấu trúc của thuật toán đó. Mẫu này thúc đẩy việc tái sử dụng mã nguồn bằng cách đưa các phần chung của thuật toán vào một lớp cha trừu tượng.

**JdbcTemplate:**
`JdbcTemplate` là một lớp trung tâm trong module Spring JDBC, giúp đơn giản hóa đáng kể việc sử dụng JDBC. Nó áp dụng mẫu thiết kế Template để xử lý tất cả các công việc lặp đi lặp lại và dễ xảy ra lỗi của JDBC, chẳng hạn như:

* Mở và đóng kết nối.
* Tạo và thực thi `PreparedStatement`.
* Lặp qua `ResultSet`.
* Xử lý `SQLException` và chuyển đổi chúng thành hệ thống phân cấp `DataAccessException` của Spring.

Việc của lập trình viên chỉ là cung cấp các phần "thay đổi" của thuật toán, bao gồm:

* Câu lệnh SQL.
* Cách ánh xạ các hàng trong `ResultSet` thành các đối tượng (thông qua các callback).

### Câu hỏi 04: Callback là gì? Ba callback interface nào của JdbcTemplate có thể được sử dụng với các truy vấn? Chúng được dùng để làm gì?

**Callback:**
Một callback là một đoạn mã hoặc một tham chiếu đến mã được truyền dưới dạng đối số cho một phương thức khác. Phương thức này sau đó sẽ "gọi lại" (execute) đoạn mã đó tại một thời điểm thích hợp. Trong Java, callback có thể được triển khai bằng các lớp implement một interface, các lớp ẩn danh (anonymous classes), hoặc từ Java 8 là các biểu thức lambda và tham chiếu phương thức (method references).

**Ba Callback Interface chính cho truy vấn trong `JdbcTemplate`:**

1.  **`RowMapper<T>`**:

    * **Mục đích**: Dùng để xử lý `ResultSet` trên cơ sở từng hàng một (per-row).
    * **Cách dùng**: Bạn triển khai phương thức `mapRow()`, phương thức này sẽ được `JdbcTemplate` gọi cho mỗi hàng trong kết quả. Bên trong `mapRow()`, bạn chỉ cần trích xuất dữ liệu từ hàng hiện tại và tạo ra một đối tượng. Bạn **không** cần gọi `ResultSet.next()`. `RowMapper` thường là stateless (không trạng thái).

2.  **`RowCallbackHandler`**:

    * **Mục đích**: Tương tự như `RowMapper`, nó xử lý dữ liệu từng hàng một.
    * **Sự khác biệt**: Phương thức `processRow()` của nó không trả về giá trị. Thay vào đó, nó thường được sử dụng khi bạn cần tích lũy kết quả trong một đối tượng bên ngoài (stateful). Ví dụ, bạn có thể tạo một danh sách và thêm các đối tượng vào đó trong mỗi lần `processRow()` được gọi.

3.  **`ResultSetExtractor<T>`**:

    * **Mục đích**: Cung cấp quyền kiểm soát hoàn toàn việc xử lý toàn bộ `ResultSet`.
    * **Cách dùng**: Bạn nhận toàn bộ `ResultSet` và bạn phải tự mình lặp qua nó (bằng cách gọi `ResultSet.next()`). Nó hữu ích cho các trường hợp ánh xạ phức tạp mà không thể xử lý trên cơ sở từng hàng, ví dụ như ánh xạ các mối quan hệ một-nhiều.

### Câu hỏi 05: Bạn có thể thực thi một câu lệnh SQL thuần túy với JDBC Template không?

**Có.** `JdbcTemplate` hoàn toàn cho phép bạn thực thi các câu lệnh SQL thuần túy. Nó cung cấp một loạt các phương thức cho mục đích này, bao gồm:

* **`query(...)`**: Dùng cho các truy vấn trả về nhiều hàng. Thường được sử dụng với một `RowMapper` hoặc `ResultSetExtractor`.
* **`queryForList(...)`**: Dùng cho các truy vấn trả về một danh sách các đối tượng. Thường là danh sách các giá trị từ một cột duy nhất hoặc danh sách các `Map` (với key là tên cột).
* **`queryForObject(...)`**: Dùng khi bạn mong đợi truy vấn trả về một đối tượng duy nhất (một hàng duy nhất). Nếu không có hàng nào hoặc có nhiều hơn một hàng, nó sẽ ném ra một ngoại lệ.
* **`queryForMap(...)`**: Dùng khi bạn mong đợi một hàng duy nhất và muốn nó được trả về dưới dạng một `Map`.
* **`execute(...)`**: Một phương thức chung để thực thi bất kỳ câu lệnh SQL nào, thường được sử dụng cho các câu lệnh DDL (Data Definition Language) như `CREATE TABLE`.
* **`update(...)`**: Dùng cho các câu lệnh DML (Data Manipulation Language) như `INSERT`, `UPDATE`, `DELETE`. Nó trả về số lượng hàng bị ảnh hưởng.

#### Ví dụ Code

```java
@Autowired
private JdbcTemplate jdbcTemplate;

public int countEmployees() {
    // Thực thi một câu lệnh SQL thuần túy để đếm số nhân viên
    String sql = "SELECT COUNT(*) FROM employees";
    return jdbcTemplate.queryForObject(sql, Integer.class);
}

public void addNewEmployee(String name, String role) {
    // Thực thi một câu lệnh INSERT thuần túy
    String sql = "INSERT INTO employees (name, role) VALUES (?, ?)";
    jdbcTemplate.update(sql, name, role);
}
```

### Câu hỏi 06: Khi nào JDBC Template lấy (và giải phóng) một kết nối? Tại sao?

Vòng đời của kết nối trong `JdbcTemplate` phụ thuộc vào việc nó có được sử dụng bên trong một giao dịch (transaction) hay không:

* **Không có giao dịch (No Transaction)**: `JdbcTemplate` sẽ lấy một kết nối từ `DataSource` **cho mỗi lần gọi phương thức** và giải phóng nó ngay sau khi phương thức hoàn thành. Lý do cho chiến lược này là để giữ tài nguyên (kết nối) trong thời gian ngắn nhất có thể, tránh lãng phí.
* **Có giao dịch (With Transaction)**: Khi được sử dụng bên trong một giao dịch do Spring quản lý (`@Transactional`), `JdbcTemplate` (thông qua `DataSourceUtils`) sẽ lấy một kết nối và **tái sử dụng kết nối đó** cho tất cả các hoạt động trong phạm vi giao dịch đó. Kết nối chỉ được giải phóng (commit/rollback và trả về pool) khi giao dịch kết thúc. Lý do là vì một giao dịch cần được thực hiện trên cùng một kết nối cơ sở dữ liệu. Việc đóng kết nối giữa chừng sẽ gây ra rollback cho các thay đổi đã thực hiện.

**Lưu ý**: Nếu `DataSource` được cấu hình với một connection pool (như HikariCP), "giải phóng" không có nghĩa là đóng kết nối vật lý, mà là trả kết nối đó về lại pool để các yêu cầu khác có thể tái sử dụng.

### Câu hỏi 07: JdbcTemplate hỗ trợ các truy vấn chung (generic queries) như thế nào? Nó trả về các đối tượng và danh sách/map của các đối tượng như thế nào?

`JdbcTemplate` hỗ trợ các truy vấn chung thông qua các phương thức đa dạng và việc sử dụng các callback.

* **`queryForObject`**: Trả về một đối tượng duy nhất. Nó mong đợi truy vấn trả về chính xác một hàng.
* **`queryForList`**: Trả về một danh sách các đối tượng. Nó mong đợi truy vấn trả về các hàng chỉ có một cột, hoặc nó sẽ trả về một danh sách các `Map`.
* **`queryForMap`**: Trả về một `Map` cho một hàng duy nhất, với key là tên cột.
* **`queryForRowSet`**: Trả về một đối tượng `SqlRowSet`, chứa metadata và cho phép lặp qua các bản ghi.

**Cách nó trả về các đối tượng và danh sách/map:**

* **Đối tượng (Objects)**: `queryForObject` sử dụng một `RowMapper` để ánh xạ một hàng thành một đối tượng tùy chỉnh. Đối với các kiểu dữ liệu cơ bản (String, Integer, ...), nó sử dụng `SingleColumnRowMapper` nội bộ.
* **Danh sách (Lists)**: `queryForList` có thể trả về một danh sách các kiểu dữ liệu cơ bản (nếu truy vấn chỉ có một cột) hoặc một danh sách các `Map`.
* **Map**: `queryForMap` sử dụng `ColumnMapRowMapper` để ánh xạ một hàng thành một `Map`.

#### Ví dụ Code: Sử dụng RowMapper

```java
public class EmployeeRowMapper implements RowMapper<Employee> {
    @Override
    public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
        Employee employee = new Employee();
        employee.setId(rs.getLong("id"));
        employee.setName(rs.getString("name"));
        employee.setRole(rs.getString("role"));
        return employee;
    }
}

// Trong DAO/Repository
public Employee findEmployeeById(long id) {
    String sql = "SELECT * FROM employees WHERE id = ?";
    // JdbcTemplate sử dụng EmployeeRowMapper để tạo đối tượng Employee từ kết quả
    return jdbcTemplate.queryForObject(sql, new EmployeeRowMapper(), id);
}
```

### Câu hỏi 08: Giao dịch là gì? Sự khác biệt giữa giao dịch cục bộ và giao dịch toàn cục là gì?

**Giao dịch (Transaction):**
Một giao dịch là một chuỗi các thao tác được coi là một đơn vị công việc duy nhất. Một giao dịch phải tuân thủ nguyên tắc **ACID**:

* **Atomicity (Tính nguyên tử)**: Tất cả các thao tác trong giao dịch đều thành công, hoặc không có thao tác nào được thực hiện.
* **Consistency (Tính nhất quán)**: Hệ thống phải chuyển từ một trạng thái hợp lệ này sang một trạng thái hợp lệ khác.
* **Isolation (Tính cô lập)**: Các giao dịch đồng thời không được ảnh hưởng lẫn nhau.
* **Durability (Tính bền vững)**: Một khi giao dịch đã được commit, các thay đổi sẽ được lưu trữ vĩnh viễn, ngay cả khi có lỗi hệ thống.

**Sự khác biệt:**

* **Giao dịch cục bộ (Local Transaction)**: Là một giao dịch chỉ liên quan đến một tài nguyên duy nhất (ví dụ: một kết nối cơ sở dữ liệu duy nhất). Chúng đơn giản hơn nhưng không thể điều phối công việc trên nhiều tài nguyên.
* **Giao dịch toàn cục (Global Transaction)**: Là một giao dịch có thể bao trùm nhiều tài nguyên giao dịch (ví dụ: hai cơ sở dữ liệu khác nhau, hoặc một cơ sở dữ liệu và một hàng đợi tin nhắn - message queue). Chúng phức tạp hơn và thường yêu cầu một trình quản lý giao dịch tuân thủ chuẩn **JTA (Java Transaction API)**, thường được cung cấp bởi các Application Server.

### Câu hỏi 09: Giao dịch có phải là một mối quan tâm xuyên suốt không? Nó được Spring triển khai như thế nào?

**Có, giao dịch là một ví dụ kinh điển về một mối quan tâm xuyên suốt.** Logic bắt đầu, commit, và rollback giao dịch cần được áp dụng cho nhiều phương thức nghiệp vụ, nhưng nó không phải là một phần của logic nghiệp vụ cốt lõi.

**Cách Spring triển khai:**
Spring triển khai quản lý giao dịch bằng cách sử dụng **AOP**. Khi bạn chú thích một phương thức hoặc một lớp bằng `@Transactional`, Spring sẽ tạo ra một proxy.

1.  Khi một phương thức được chú thích `@Transactional` được gọi, lời gọi sẽ bị chặn bởi proxy.
2.  Proxy (cụ thể là `TransactionInterceptor`) sẽ tương tác với một `PlatformTransactionManager` để bắt đầu một giao dịch mới hoặc tham gia vào một giao dịch hiện có.
3.  Phương thức nghiệp vụ gốc được thực thi.
4.  Nếu phương thức thực thi thành công, proxy sẽ yêu cầu `PlatformTransactionManager` commit giao dịch.
5.  Nếu phương thức ném ra một ngoại lệ (mặc định là `RuntimeException` hoặc `Error`), proxy sẽ yêu cầu `PlatformTransactionManager` rollback giao dịch.

### Câu hỏi 10: Bạn sẽ định nghĩa một giao dịch trong Spring như thế nào? @Transactional làm gì? PlatformTransactionManager là gì?

**Cách định nghĩa một giao dịch trong Spring:**

1.  **Kích hoạt quản lý giao dịch**: Thêm annotation `@EnableTransactionManagement` vào một lớp `@Configuration`.
2.  **Cung cấp một `PlatformTransactionManager`**: Tạo một bean triển khai interface `PlatformTransactionManager`. Các implementation phổ biến bao gồm:
    * `DataSourceTransactionManager`: Cho JDBC.
    * `JpaTransactionManager`: Cho JPA.
    * `JtaTransactionManager`: Cho các giao dịch toàn cục JTA.
3.  **Khai báo giao dịch**: Sử dụng annotation `@Transactional` trên các lớp hoặc phương thức cần được thực thi trong một giao dịch.

**`@Transactional` làm gì?**
`@Transactional` là một annotation khai báo (declarative) để chỉ định rằng một phương thức (hoặc tất cả các phương thức trong một lớp) nên được bao bọc trong một giao dịch. Nó hướng dẫn AOP của Spring áp dụng advice về giao dịch. Annotation này có nhiều thuộc tính để tùy chỉnh hành vi của giao dịch, chẳng hạn như:

* `propagation`: Cách giao dịch tương tác với một giao dịch hiện có (ví dụ: `REQUIRED`, `REQUIRES_NEW`).
* `isolation`: Mức độ cô lập của giao dịch (ví dụ: `READ_COMMITTED`, `SERIALIZABLE`).
* `readOnly`: Gợi ý cho trình điều khiển cơ sở dữ liệu để tối ưu hóa cho các hoạt động chỉ đọc.
* `timeout`: Thời gian tối đa mà giao dịch có thể chạy trước khi bị rollback.
* `rollbackFor`, `noRollbackFor`: Tùy chỉnh các ngoại lệ sẽ gây ra hoặc không gây ra rollback.

**`PlatformTransactionManager` là gì?**
Đây là một interface trung tâm trong cơ chế quản lý giao dịch của Spring. Nó trừu tượng hóa việc quản lý giao dịch khỏi các API cụ thể. Vai trò của nó là xử lý vòng đời của giao dịch:

* `getTransaction(...)`: Lấy giao dịch hiện tại hoặc tạo một giao dịch mới.
* `commit(...)`: Commit giao dịch.
* `rollback(...)`: Rollback giao dịch.

Bằng cách sử dụng interface này, mã nguồn của bạn trở nên độc lập với cách triển khai giao dịch cụ thể (JDBC, JPA, JTA).

### Quản lý Giao dịch Nâng cao trong Spring

Phần này khám phá các khía cạnh chi tiết và các tình huống phức tạp hơn trong việc quản lý giao dịch với Spring.

### Câu hỏi 11: JdbcTemplate có thể tham gia vào một giao dịch hiện có không?

**Có.** `JdbcTemplate` hoàn toàn có khả năng tham gia vào một giao dịch hiện có. Nó sẽ hỗ trợ cả giao dịch được tạo ra một cách khai báo (declarative) bằng annotation `@Transactional` và giao dịch được tạo ra một cách lập trình (programmatic).

Khi một phương thức `JdbcTemplate` được gọi trong phạm vi của một giao dịch đang hoạt động, nó sẽ sử dụng cùng một kết nối cơ sở dữ liệu đã được liên kết với giao dịch đó, đảm bảo rằng tất cả các hoạt động cơ sở dữ liệu đều là một phần của cùng một đơn vị công việc (unit of work).

#### Ví dụ Code

```java
@Service
public class OrderService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private AuditService auditService; // Một service khác cũng có thể sử dụng JdbcTemplate

    @Transactional
    public void createOrder(Order order) {
        // Thao tác 1: Lưu đơn hàng
        String sqlOrder = "INSERT INTO orders (id, product, quantity) VALUES (?, ?, ?)";
        jdbcTemplate.update(sqlOrder, order.getId(), order.getProduct(), order.getQuantity());

        // Gọi một phương thức khác cũng tham gia vào cùng giao dịch này
        auditService.logAction("CREATED_ORDER", order.getId());

        // Nếu có lỗi xảy ra ở đây, cả việc lưu đơn hàng và ghi log sẽ bị rollback
    }
}

@Service
public class AuditService {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    // Propagation mặc định là REQUIRED, nên nó sẽ tham gia vào giao dịch hiện có của createOrder
    @Transactional
    public void logAction(String action, String entityId) {
        String sqlAudit = "INSERT INTO audit_log (action, entity_id, log_time) VALUES (?, ?, ?)";
        jdbcTemplate.update(sqlAudit, action, entityId, new Date());
    }
}
```

### Câu hỏi 12: Mức độ cô lập giao dịch là gì? Chúng ta có bao nhiêu mức và chúng được sắp xếp như thế nào?

**Mức độ cô lập giao dịch (Transaction Isolation Level)** xác định cách mà các thay đổi được thực hiện trong một giao dịch này hiển thị đối với các giao dịch đồng thời khác và những người dùng khác của hệ thống.

* **Mức độ cô lập cao hơn** có nghĩa là các thay đổi từ một giao dịch ít có khả năng hiển thị hơn, đảm bảo tính nhất quán dữ liệu cao hơn nhưng có thể làm giảm thông lượng và tính đồng thời của hệ thống.
* **Mức độ cô lập thấp hơn** có nghĩa là các thay đổi từ một giao dịch có thể "lọt" vào các câu lệnh `select` được thực thi trong các giao dịch khác, làm cho dữ liệu kém nhất quán hơn nhưng tăng thông lượng.

**Ba vấn đề chính có thể xảy ra do mức độ cô lập:**

1.  **Dirty Read**: Một giao dịch đọc dữ liệu chưa được commit bởi một giao dịch khác.
2.  **Non-repeatable Read**: Một giao dịch đọc lại cùng một hàng dữ liệu nhưng nhận được kết quả khác nhau vì một giao dịch khác đã cập nhật hàng đó.
3.  **Phantom Read**: Một giao dịch thực thi lại một truy vấn và thấy có thêm các hàng mới được chèn vào bởi một giao dịch khác.

**Các mức độ cô lập (từ thấp đến cao):**

1.  **READ\_UNCOMMITTED**: Cho phép Dirty Read, Non-repeatable Read và Phantom Read.
2.  **READ\_COMMITTED**: Ngăn chặn Dirty Read. Đây là mức mặc định của hầu hết các cơ sở dữ liệu (ví dụ: PostgreSQL, SQL Server, Oracle).
3.  **REPEATABLE\_READ**: Ngăn chặn Dirty Read và Non-repeatable Read. Đây là mức mặc định của MySQL.
4.  **SERIALIZABLE**: Ngăn chặn cả ba vấn đề. Đây là mức cô lập cao nhất, đảm bảo các giao dịch được thực thi tuần tự, nhưng ảnh hưởng nhiều nhất đến hiệu suất.

### Câu hỏi 13: @EnableTransactionManagement dùng để làm gì?

Annotation `@EnableTransactionManagement` được sử dụng trên một lớp `@Configuration` để **kích hoạt quản lý giao dịch dựa trên annotation** (`@Transactional`) trong Spring Framework.

Khi `@EnableTransactionManagement` được sử dụng, Spring sẽ tự động áp dụng `TransactionInterceptor` và `TransactionAspectSupport` để tạo proxy cho mỗi lời gọi đến một lớp hoặc phương thức được chú thích `@Transactional`. Các proxy này sẽ sử dụng `PlatformTransactionManager` để quản lý vòng đời của giao dịch.

`@EnableTransactionManagement` cho phép bạn chỉ định các giá trị sau:

* `mode`: Đặt chế độ advice cho annotation `@Transactional`. Mặc định là `PROXY`, bạn có thể chuyển sang `ASPECTJ` để có weaving nâng cao hơn.
* `order`: Chỉ định thứ tự thực thi của advice khi có nhiều advice áp dụng cho cùng một join point.
* `proxyTargetClass`: Chỉ định liệu nên tạo proxy CGLIB (true) hay proxy JDK (false - mặc định).

### Câu hỏi 14: Transaction propagation có nghĩa là gì?

**Transaction Propagation (Sự lan truyền giao dịch)** định nghĩa hành vi của một phương thức giao dịch khi nó được gọi từ bên trong một phương thức khác cũng có giao dịch. Nó xác định xem phương thức được gọi có nên tham gia vào giao dịch hiện có, tạo một giao dịch mới, hay thực thi mà không có giao dịch.

Bạn có thể định nghĩa sự lan truyền trong thuộc tính `propagation` của annotation `@Transactional`. Các tùy chọn bao gồm:

* **`REQUIRED` (mặc định)**: Hỗ trợ một giao dịch hiện tại, tạo một giao dịch mới nếu chưa có.
* **`SUPPORTS`**: Hỗ trợ một giao dịch hiện tại, thực thi không có giao dịch nếu chưa có.
* **`MANDATORY`**: Hỗ trợ một giao dịch hiện tại, ném ra ngoại lệ nếu chưa có.
* **`REQUIRES_NEW`**: Luôn tạo một giao dịch mới, và tạm ngưng giao dịch hiện tại (nếu có).
* **`NOT_SUPPORTED`**: Thực thi không có giao dịch, tạm ngưng giao dịch hiện tại (nếu có).
* **`NEVER`**: Thực thi không có giao dịch, ném ra ngoại lệ nếu có một giao dịch đang tồn tại.
* **`NESTED`**: Thực thi bên trong một giao dịch lồng nhau (nested transaction) nếu có một giao dịch hiện tại, hoạt động giống như `REQUIRED` nếu không có.

### Câu hỏi 15: Điều gì xảy ra nếu một phương thức được chú thích @Transactional gọi một phương thức khác được chú thích @Transactional trên cùng một đối tượng?

**Không có gì xảy ra.** Lời gọi phương thức thứ hai sẽ không được thực thi trong một ngữ cảnh giao dịch mới hoặc tham gia vào giao dịch hiện có theo cách bạn mong đợi.

**Lý do:** Các proxy của Spring (cả JDK và CGLIB) không hỗ trợ **tự gọi (self-invocation)**. Khi bạn gọi một phương thức từ bên trong cùng một lớp, lời gọi đó không đi qua proxy mà đi trực tiếp đến đối tượng gốc. Do đó, `TransactionInterceptor` không có cơ hội chặn lời gọi và áp dụng logic giao dịch cho phương thức thứ hai.

#### Ví dụ Code minh họa vấn đề

```java
@Service
public class SelfInvocationService {

    @Transactional
    public void methodA() {
        // ... thực hiện một số công việc ...
        
        // Lời gọi này sẽ KHÔNG đi qua proxy
        // Do đó, các thiết lập @Transactional của methodB sẽ bị bỏ qua
        this.methodB(); 
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void methodB() {
        // Giao dịch mới sẽ không được tạo ra ở đây
        // ... thực hiện công việc khác ...
    }
}
```

### Câu hỏi 16: Annotation @Transactional có thể được sử dụng ở đâu? Cách sử dụng điển hình nếu bạn đặt nó ở cấp độ lớp là gì?

`@Transactional` có thể được sử dụng ở hai cấp độ:

1.  **Cấp độ phương thức (Method level)**: Áp dụng các cài đặt giao dịch chỉ cho phương thức đó.
2.  **Cấp độ lớp (Class level)**: Áp dụng các cài đặt giao dịch mặc định cho tất cả các phương thức `public` trong lớp đó.

**Cách sử dụng điển hình ở cấp độ lớp:**
Một cách tiếp cận phổ biến là đặt `@Transactional(readOnly = true)` ở cấp độ lớp Repository hoặc Service. Điều này thiết lập rằng, theo mặc định, tất cả các phương thức là các giao dịch chỉ đọc, đây là một tối ưu hóa tốt. Sau đó, bạn có thể ghi đè cài đặt này trên các phương thức cần ghi dữ liệu (ví dụ: `save`, `update`, `delete`) bằng cách chú thích chúng với `@Transactional` (không có `readOnly = true`).

#### Ví dụ Code

```java
@Service
@Transactional(readOnly = true) // Mặc định tất cả các phương thức là chỉ đọc
public class ProductService {

    public Product findProductById(Long id) {
        // Phương thức này sẽ thừa hưởng giao dịch readOnly = true từ lớp
        // ... logic tìm kiếm sản phẩm ...
    }
    
    // Ghi đè cài đặt mặc định của lớp cho phương thức này
    @Transactional 
    public void saveProduct(Product product) {
        // Phương thức này sẽ có một giao dịch đọc-ghi
        // ... logic lưu sản phẩm ...
    }
}
```

### Câu hỏi 17: Quản lý giao dịch khai báo có nghĩa là gì?

**Quản lý giao dịch khai báo (Declarative Transaction Management)** có nghĩa là bạn tách biệt logic quản lý giao dịch khỏi logic nghiệp vụ của mình bằng cách sử dụng các annotation (như `@Transactional`) hoặc cấu hình XML.

Thay vì viết mã một cách tường minh để bắt đầu, commit, và rollback giao dịch (được gọi là quản lý giao dịch lập trình - programmatic transaction management), bạn chỉ cần "khai báo" rằng một phương thức hoặc một lớp nên có hành vi giao dịch. Framework (Spring, thông qua AOP) sẽ lo phần còn lại.

**Ưu điểm:**

* **Mã nguồn sạch sẽ hơn**: Logic nghiệp vụ không bị lộn xộn với mã quản lý giao dịch lặp đi lặp lại.
* **Giảm lỗi**: Giảm thiểu nguy cơ quên đóng kết nối hoặc xử lý rollback không đúng cách.
* **Dễ bảo trì**: Dễ dàng thay đổi các thuộc tính giao dịch mà không cần sửa đổi mã nghiệp vụ.

### Câu hỏi 18: Chính sách rollback mặc định là gì? Làm thế nào bạn có thể ghi đè nó?

**Chính sách rollback mặc định** của Spring là tự động rollback một giao dịch chỉ khi một **unchecked exception** (một ngoại lệ kế thừa từ `RuntimeException`) hoặc một `Error` được ném ra từ phương thức được chú thích `@Transactional`.

Các **checked exceptions** (ngoại lệ kế thừa từ `Exception` nhưng không phải `RuntimeException`) sẽ **không** gây ra rollback theo mặc định.

**Cách ghi đè:**
Bạn có thể ghi đè hành vi này bằng cách sử dụng các thuộc tính `rollbackFor` và `noRollbackFor` trong annotation `@Transactional`.

* **`rollbackFor`**: Chỉ định một mảng các lớp ngoại lệ sẽ gây ra rollback.
* **`noRollbackFor`**: Chỉ định một mảng các lớp ngoại lệ sẽ **không** gây ra rollback, ngay cả khi chúng là unchecked exceptions.

#### Ví dụ Code

```java
@Service
public class OrderProcessingService {

    // Rollback khi có CustomCheckedException, mặc dù nó là một checked exception
    @Transactional(rollbackFor = CustomCheckedException.class)
    public void processOrder(Order order) throws CustomCheckedException {
        // ... logic xử lý ...
        if (orderIsInvalid(order)) {
            throw new CustomCheckedException("Đơn hàng không hợp lệ!");
        }
    }
    
    // Không rollback khi có IgnorableException, mặc dù nó là một unchecked exception
    @Transactional(noRollbackFor = IgnorableException.class)
    public void updateOrderStatus(Long orderId, String status) {
        // ... logic cập nhật ...
        if (status.equals("IGNORED")) {
            throw new IgnorableException("Trạng thái bị bỏ qua, không cần rollback.");
        }
    }
}

// Một checked exception tùy chỉnh
class CustomCheckedException extends Exception {
    public CustomCheckedException(String message) { super(message); }
}

// Một unchecked exception tùy chỉnh
class IgnorableException extends RuntimeException {
    public IgnorableException(String message) { super(message); }
}
```

### Câu hỏi 19: Chính sách rollback mặc định trong một bài test JUnit là gì, khi bạn sử dụng Spring Test và chú thích phương thức @Test bằng @Transactional?

Trong một bài test JUnit được tích hợp với Spring (`@RunWith(SpringRunner.class)` hoặc `@ExtendWith(SpringExtension.class)`) và có một phương thức `@Test` được chú thích bằng `@Transactional`, **chính sách mặc định là luôn luôn rollback**.

Điều này có nghĩa là sau khi phương thức test kết thúc, mọi thay đổi được thực hiện đối với cơ sở dữ liệu trong phạm vi giao dịch của bài test sẽ được hoàn tác.

**Lý do:** Điều này đảm bảo rằng các bài test độc lập với nhau. Mỗi bài test chạy trên một "bảng trắng sạch sẽ" và không để lại bất kỳ dữ liệu rác nào có thể ảnh hưởng đến các bài test sau đó.

**Làm thế nào để ghi đè:**
Nếu bạn thực sự muốn commit các thay đổi trong một bài test (thường không được khuyến khích), bạn có thể thêm annotation `@Commit` vào phương thức `@Test` đó. Ngược lại, bạn cũng có thể sử dụng `@Rollback(false)`.

### Câu hỏi 20: Tại sao thuật ngữ "đơn vị công việc" lại quan trọng và tại sao JDBC AutoCommit lại vi phạm mẫu này?

**Tầm quan trọng của "Đơn vị công việc" (Unit of Work):**
"Đơn vị công việc" là một thuật ngữ chung để mô tả một tập hợp các tác vụ thực hiện một số thay đổi trên dữ liệu, với giả định rằng **tất cả các thay đổi phải được thực hiện, hoặc không có thay đổi nào được thực hiện cả**. Đây chính là bản chất của tính nguyên tử (Atomicity) trong nguyên tắc ACID. Trong cơ sở dữ liệu quan hệ, một đơn vị công việc được thể hiện bằng một Giao dịch Cơ sở dữ liệu.

Thuật ngữ này rất quan trọng vì nó đảm bảo tính toàn vẹn và nhất quán của dữ liệu. Nếu một chuỗi các thao tác liên quan (ví dụ: chuyển tiền từ tài khoản A sang tài khoản B) không được xử lý như một đơn vị công việc duy nhất, hệ thống có thể rơi vào trạng thái không hợp lệ (ví dụ: tiền đã bị trừ khỏi A nhưng chưa được cộng vào B).

**Tại sao JDBC AutoCommit vi phạm mẫu này?**
Chế độ `AutoCommit` của JDBC, khi được bật (thường là mặc định), sẽ coi mỗi câu lệnh SQL là một giao dịch riêng biệt và commit nó ngay lập tức sau khi thực thi.

Điều này vi phạm mẫu "đơn vị công việc" vì nó không cho phép bạn nhóm nhiều câu lệnh SQL liên quan vào một giao dịch duy nhất. Nếu bạn có hai câu lệnh `UPDATE` cần phải thành công cùng nhau, với `AutoCommit=true`, nếu câu lệnh thứ hai thất bại, câu lệnh đầu tiên đã được commit và không thể được rollback, dẫn đến dữ liệu không nhất quán. Đó là lý do tại sao các framework như Spring luôn tắt `AutoCommit` khi quản lý giao dịch.

### Tích hợp Spring với JPA (Spring Data JPA)

Phần này sẽ tập trung vào cách Spring tích hợp với Java Persistence API (JPA) để đơn giản hóa tầng truy cập dữ liệu.

### Câu hỏi 21: Bạn cần làm gì trong Spring nếu muốn làm việc với JPA thuần (plain JPA)?

Để làm việc với JPA thuần trong Spring, bạn cần phải tương tác trực tiếp với hai thành phần cốt lõi của JPA: **`EntityManagerFactory`** và **`EntityManager`**.

1.  **Cấu hình `EntityManagerFactory`**: Bạn cần định nghĩa một bean `EntityManagerFactory` trong cấu hình Spring của mình. Bean này chịu trách nhiệm tạo ra các `EntityManager`.
2.  **Lấy và sử dụng `EntityManager`**: Trong các lớp DAO/Repository của mình, bạn cần tiêm (`@PersistenceContext`) một `EntityManager`. `EntityManager` là interface chính để tương tác với persistence context, cho phép bạn thực hiện các thao tác CRUD (Create, Read, Update, Delete) trên các entity.

Cách tiếp cận này cho phép bạn sử dụng đầy đủ các tính năng của JPA nhưng đòi hỏi bạn phải viết khá nhiều mã lặp đi lặp lại để quản lý `EntityManager` và xử lý các ngoại lệ.

#### Ví dụ Code: Repository sử dụng JPA thuần

```java
@Repository // Đánh dấu là một Spring bean cho tầng truy cập dữ liệu
public class PlainJpaProductDao {

    // Tiêm EntityManager để tương tác với persistence context
    @PersistenceContext
    private EntityManager entityManager;

    public Product findById(Long id) {
        return entityManager.find(Product.class, id);
    }

    public void save(Product product) {
        entityManager.persist(product);
    }

    public void update(Product product) {
        entityManager.merge(product);
    }

    public void delete(Product product) {
        entityManager.remove(entityManager.contains(product) ? product : entityManager.merge(product));
    }
}
```

-----

### Câu hỏi 22: Bạn có thể tham gia vào một giao dịch hiện có trong Spring khi sử dụng JPA thuần không?

**Có.** Khi bạn tiêm một `EntityManager` bằng annotation `@PersistenceContext`, bạn sẽ nhận được một proxy của `EntityManager` có khả năng nhận biết giao dịch (transaction-aware).

Nếu có một giao dịch do Spring quản lý (`@Transactional`) đang hoạt động, `EntityManager` này sẽ tự động tham gia vào giao dịch đó. Điều này có nghĩa là tất cả các thao tác được thực hiện thông qua `EntityManager` sẽ là một phần của cùng một đơn vị công việc. Nếu không có giao dịch nào đang hoạt động, mỗi lời gọi đến `EntityManager` sẽ được thực thi trong một giao dịch ngắn riêng của nó.

-----

### Câu hỏi 23: Bạn có thể sử dụng (các) PlatformTransactionManager nào với JPA?

Khi làm việc với JPA, bạn có hai lựa chọn chính cho `PlatformTransactionManager`:

1.  **`JpaTransactionManager`**:

    * Đây là lựa chọn phổ biến nhất và được khuyên dùng cho các ứng dụng sử dụng một `DataSource` duy nhất.
    * Nó hoạt động trực tiếp với một `EntityManagerFactory` để quản lý các giao dịch trên một cơ sở dữ liệu duy nhất (giao dịch cục bộ).

2.  **`JtaTransactionManager`**:

    * Được sử dụng cho các giao dịch toàn cục (global transactions) hoặc giao dịch phân tán, khi bạn cần điều phối các thao tác trên nhiều tài nguyên giao dịch khác nhau (ví dụ: hai cơ sở dữ liệu khác nhau, hoặc một cơ sở dữ liệu và một message queue).
    * Nó ủy quyền việc quản lý giao dịch cho một trình quản lý giao dịch tuân thủ chuẩn JTA (Java Transaction API), thường được cung cấp bởi các Application Server như WildFly, WebSphere.

-----

### Câu hỏi 24: Bạn phải cấu hình những gì để sử dụng JPA với Spring và làm cho nó hoạt động?

Để thiết lập JPA với Spring, bạn cần cấu hình các bean sau:

1.  **`DataSource`**: Bean này cung cấp các kết nối đến cơ sở dữ liệu.
2.  **`EntityManagerFactory`**: Đây là bean trung tâm của JPA. Spring cung cấp lớp `LocalContainerEntityManagerFactoryBean` để dễ dàng cấu hình nó. Bạn cần cung cấp `DataSource`, nhà cung cấp JPA (JPA vendor, ví dụ: Hibernate), và vị trí của các lớp entity.
3.  **`PlatformTransactionManager`**: Cần một bean `JpaTransactionManager` để kích hoạt quản lý giao dịch khai báo (`@Transactional`).
4.  **Kích hoạt Quản lý Giao dịch**: Thêm annotation `@EnableTransactionManagement` vào lớp cấu hình của bạn.
5.  **(Tùy chọn nhưng được khuyên dùng)** Kích hoạt Dịch thuật Ngoại lệ (Exception Translation): Thêm annotation `@Repository` vào các lớp DAO của bạn. Điều này cho phép `PersistenceExceptionTranslationPostProcessor` của Spring bắt các ngoại lệ dành riêng cho nhà cung cấp JPA và chuyển đổi chúng thành hệ thống phân cấp `DataAccessException` nhất quán của Spring.

#### Ví dụ Code: Cấu hình đầy đủ

```java
@Configuration
@EnableTransactionManagement
public class JpaConfig {

    // 1. Cấu hình DataSource
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:db;DB_CLOSE_DELAY=-1");
        dataSource.setUsername("sa");
        dataSource.setPassword("sa");
        return dataSource;
    }

    // 2. Cấu hình EntityManagerFactory
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(dataSource());
        // Chỉ định package chứa các entity để JPA quét
        em.setPackagesToScan(new String[] { "com.example.domain" });

        // Chỉ định nhà cung cấp JPA là Hibernate
        JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        em.setJpaVendorAdapter(vendorAdapter);
        em.setJpaProperties(additionalProperties());

        return em;
    }

    // 3. Cấu hình Trình quản lý Giao dịch
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(emf);
        return transactionManager;
    }

    // 5. Kích hoạt Dịch thuật Ngoại lệ (hoạt động khi có @Repository)
    @Bean
    public PersistenceExceptionTranslationPostProcessor exceptionTranslation(){
        return new PersistenceExceptionTranslationPostProcessor();
    }

    // Các thuộc tính bổ sung cho Hibernate
    Properties additionalProperties() {
        Properties properties = new Properties();
        properties.setProperty("hibernate.hbm2ddl.auto", "create-drop");
        properties.setProperty("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
        return properties;
    }
}
```

-----

### Câu hỏi 25: Repository interface là gì?

Trong bối cảnh của Spring Data, một **Repository interface** là một interface mà bạn định nghĩa để khai báo các thao tác truy cập dữ liệu cho một entity cụ thể. Spring Data sau đó sẽ tự động cung cấp một implementation cho interface này tại thời điểm chạy.

Mục đích chính của nó là giảm đáng kể lượng mã lặp đi lặp lại cần thiết để triển khai các tầng truy cập dữ liệu. Bằng cách kế thừa từ các interface cơ sở của Spring Data (như `CrudRepository` hoặc `JpaRepository`), bạn sẽ có ngay các phương thức CRUD cơ bản mà không cần viết bất kỳ dòng mã triển khai nào.

-----

### Câu hỏi 26: Bạn định nghĩa một Repository interface như thế nào? Tại sao nó là một interface mà không phải là một lớp?

**Cách định nghĩa:**
Bạn định nghĩa một repository bằng cách tạo một interface và cho nó kế thừa từ một trong các interface cơ sở của Spring Data, phổ biến nhất là `JpaRepository<T, ID>`.

* **`T`**: Là kiểu của entity mà repository này quản lý.
* **`ID`**: Là kiểu của khóa chính (primary key) của entity đó.

**Tại sao là interface?**
Nó được định nghĩa là một interface vì bạn chỉ cần **khai báo** các phương thức truy cập dữ liệu mà bạn muốn. Bạn không cần phải viết logic triển khai. Spring Data sẽ đảm nhận việc đó.

Việc sử dụng interface cho phép Spring:

1.  **Tạo proxy tại thời điểm chạy**: Spring sẽ quét các interface kế thừa từ `Repository`, phân tích cú pháp tên các phương thức bạn đã định nghĩa, và tạo ra một đối tượng proxy triển khai các phương thức đó với logic truy vấn tương ứng.
2.  **Tách biệt hoàn toàn**: Tách biệt hoàn toàn "cái gì" (what - các thao tác bạn muốn) khỏi "cách làm" (how - logic triển khai cụ thể). Điều này giúp mã nguồn của bạn sạch sẽ và tập trung vào nghiệp vụ.

#### Ví dụ Code

```java
// Định nghĩa một interface kế thừa từ JpaRepository
// Spring Data sẽ tự động triển khai nó
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    // Bạn không cần viết mã cho các phương thức như save(), findById(), findAll(), ...
    // Chúng đã có sẵn từ JpaRepository.
}
```

-----

### Câu hỏi 27: Quy tắc đặt tên cho các phương thức finder trong một Repository interface là gì?

Spring Data có một cơ chế mạnh mẽ để tự động tạo các truy vấn từ tên phương thức. Bạn chỉ cần đặt tên phương thức theo một quy ước nhất định, và Spring Data sẽ phân tích cú pháp tên đó để tạo ra câu lệnh truy vấn JPA (JPQL) tương ứng.

**Quy tắc chung:**
`find...By...`, `read...By...`, `query...By...`, `count...By...`, `get...By...`

**Các từ khóa (Keywords) phổ biến:**

* **`And`**: `findByNameAndCategory(String name, String category)` -\> `... WHERE x.name = ?1 AND x.category = ?2`
* **`Or`**: `findByNameOrCategory(String name, String category)` -\> `... WHERE x.name = ?1 OR x.category = ?2`
* **`Is`**, **`Equals`**: `findByNameIs(String name)` -\> `... WHERE x.name = ?1`
* **`Between`**: `findByStartDateBetween(Date start, Date end)` -\> `... WHERE x.startDate BETWEEN ?1 AND ?2`
* **`LessThan`**, **`GreaterThan`**: `findByPriceLessThan(double price)` -\> `... WHERE x.price < ?1`
* **`IsNull`**, **`IsNotNull`**: `findByDescriptionIsNull()` -\> `... WHERE x.description IS NULL`
* **`Like`**, **`NotLike`**: `findByNameLike(String pattern)` -\> `... WHERE x.name LIKE ?1`
* **`StartingWith`**, **`EndingWith`**, **`Containing`**: `findByNameStartingWith(String prefix)` -\> `... WHERE x.name LIKE ?1` (với `?1` là `prefix%`)
* **`OrderBy`**: `findByCategoryOrderByPriceAsc(String category)` -\> `... WHERE x.category = ?1 ORDER BY x.price ASC`
* **`Distinct`**: `findDistinctByCategory(String category)` -\> `SELECT DISTINCT ...`

#### Ví dụ Code

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // SELECT u FROM User u WHERE u.emailAddress = ?1
    User findByEmailAddress(String emailAddress);

    // SELECT u FROM User u WHERE u.lastName = ?1 ORDER BY u.firstName ASC
    List<User> findByLastNameOrderByFirstNameAsc(String lastName);

    // SELECT u FROM User u WHERE u.age > ?1
    List<User> findByAgeGreaterThan(int age);
}
```

-----

### Câu hỏi 28: Các Spring Data repository được Spring triển khai như thế nào tại thời điểm chạy?

Tại thời điểm chạy, Spring Data thực hiện các bước sau để tạo ra implementation cho các repository interface của bạn:

1.  **Quét (Scanning)**: Spring quét classpath để tìm các interface kế thừa từ `Repository` (hoặc các interface con của nó như `JpaRepository`). Việc này được kích hoạt bởi annotation `@EnableJpaRepositories`.
2.  **Tạo Proxy**: Đối với mỗi interface tìm thấy, Spring sử dụng một `FactoryBean` (cụ thể là `JpaRepositoryFactoryBean`) để tạo ra một đối tượng proxy động (JDK Dynamic Proxy).
3.  **Thêm các Implementation cơ bản**: Proxy này được "trộn" với một implementation cơ bản (ví dụ: `SimpleJpaRepository`) cung cấp logic cho các phương thức CRUD chuẩn (như `save`, `findById`, `findAll`).
4.  **Phân tích cú pháp các phương thức tùy chỉnh**: Proxy cũng phân tích tên của các phương thức finder mà bạn đã định nghĩa trong interface.
5.  **Tạo truy vấn**: Dựa trên quy tắc đặt tên, nó tạo ra các câu lệnh JPQL tương ứng.
6.  **Thực thi truy vấn**: Khi một phương thức finder được gọi trên proxy, nó sẽ sử dụng `EntityManager` để thực thi câu lệnh JPQL đã được tạo ra và trả về kết quả.

Về cơ bản, Spring Data đóng vai trò là một "nhà máy" tạo ra các lớp DAO/Repository cho bạn một cách tự động, giúp bạn tiết kiệm rất nhiều công sức.

-----

### Câu hỏi 29: @Query được dùng để làm gì?

Annotation `@Query` được sử dụng trên một phương thức trong repository interface để cung cấp một câu lệnh truy vấn tùy chỉnh một cách tường minh. Điều này rất hữu ích trong các trường hợp sau:

1.  **Truy vấn quá phức tạp**: Khi quy tắc đặt tên phương thức không đủ để biểu diễn một truy vấn phức tạp (ví dụ: có các phép `JOIN` phức tạp, các hàm tổng hợp, hoặc các truy vấn con).
2.  **Tên phương thức quá dài**: Khi tên phương thức trở nên quá dài và khó đọc nếu tuân theo quy tắc đặt tên.
3.  **Sử dụng SQL thuần (Native SQL)**: Khi bạn cần thực thi một câu lệnh SQL dành riêng cho cơ sở dữ liệu của mình mà không thể biểu diễn bằng JPQL.

`@Query` có thể nhận một câu lệnh **JPQL** (mặc định) hoặc một câu lệnh **SQL thuần** (bằng cách đặt thuộc tính `nativeQuery = true`).

#### Ví dụ Code

```java
public interface InvoiceRepository extends JpaRepository<Invoice, Long> {

    // Ví dụ sử dụng JPQL với các tham số được đặt tên
    @Query("SELECT i FROM Invoice i WHERE i.amount > :minAmount AND i.customer.name = :customerName")
    List<Invoice> findLargeInvoicesForCustomer(@Param("minAmount") BigDecimal amount, @Param("customerName") String name);

    // Ví dụ sử dụng SQL thuần để gọi một hàm dành riêng cho CSDL
    @Query(value = "SELECT * FROM invoices WHERE created_date > CURRENT_DATE - INTERVAL '7 days'", nativeQuery = true)
    List<Invoice> findRecentInvoicesNative();
}
```