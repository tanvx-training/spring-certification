### Lập Trình Hướng Khía Cạnh (AOP) trong Spring

Phần này tập trung vào các khái niệm cốt lõi của AOP, một trong những tính năng mạnh mẽ nhất của Spring.

### Câu hỏi 01: Khái niệm về AOP là gì? Nó giải quyết vấn đề gì?

**AOP (Aspect-Oriented Programming - Lập trình hướng khía cạnh)** là một mô hình lập trình bổ sung cho Lập trình hướng đối tượng (OOP) bằng cách cung cấp một cách để tách biệt các **mối quan tâm xuyên suốt (cross-cutting concerns)** khỏi logic nghiệp vụ của mã nguồn. Điều này đạt được bằng khả năng thêm hành vi bổ sung vào mã nguồn mà không cần phải sửa đổi chính mã nguồn đó.

**Vấn đề mà AOP giải quyết:**
AOP giải quyết vấn đề của các mối quan tâm xuyên suốt. Đây là những chức năng cần được áp dụng tại nhiều điểm trong một ứng dụng, chẳng hạn như:

* **Logging (Ghi log)**: Ghi lại thông tin về các hoạt động của hệ thống.
* **Security (Bảo mật)**: Kiểm tra quyền truy cập trước khi thực thi một phương thức.
* **Transactions (Giao dịch)**: Quản lý các giao dịch cơ sở dữ liệu (bắt đầu, commit, rollback).

Nếu không sử dụng AOP, bạn sẽ gặp phải hai vấn đề chính:

1.  **Trùng lặp mã nguồn (Code Duplication)**: Logic cho các mối quan tâm này (ví dụ: mã ghi log) sẽ bị lặp lại ở đầu và cuối của nhiều phương thức.
2.  **Trộn lẫn các mối quan tâm (Mixing of Concerns)**: Logic nghiệp vụ bị trộn lẫn với logic kỹ thuật (như quản lý giao dịch), làm cho mã nguồn trở nên khó đọc và khó bảo trì.

#### Ví dụ Code: Vấn đề và Giải pháp

**Không sử dụng AOP (Mã nguồn bị trộn lẫn và trùng lặp):**

```java
public class OrderService {
    public void placeOrder(Order order) {
        System.out.println("LOG: Bắt đầu phương thức placeOrder..."); // Mối quan tâm ghi log
        // Bắt đầu giao dịch...
        
        // ... Logic nghiệp vụ chính để đặt hàng ...
        
        // Commit giao dịch...
        System.out.println("LOG: Kết thúc phương thức placeOrder."); // Mối quan tâm ghi log
    }

    public void cancelOrder(String orderId) {
        System.out.println("LOG: Bắt đầu phương thức cancelOrder..."); // Mối quan tâm ghi log
        // Bắt đầu giao dịch...

        // ... Logic nghiệp vụ chính để hủy đơn hàng ...

        // Commit giao dịch...
        System.out.println("LOG: Kết thúc phương thức cancelOrder."); // Mối quan tâm ghi log
    }
}
```

**Sử dụng AOP (Tách biệt các mối quan tâm):**

Mã nguồn nghiệp vụ trở nên sạch sẽ hơn.

```java
@Service
public class OrderService {
    public void placeOrder(Order order) {
        // Chỉ chứa logic nghiệp vụ
        System.out.println("Đang xử lý đặt hàng cho: " + order.getId());
    }

    public void cancelOrder(String orderId) {
        // Chỉ chứa logic nghiệp vụ
        System.out.println("Đang xử lý hủy đơn hàng: " + orderId);
    }
}
```

Logic ghi log được tách ra một "khía cạnh" (Aspect) riêng.

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.OrderService.*(..))")
    public void logBeforeMethodCall(JoinPoint joinPoint) {
        System.out.println("LOG: Bắt đầu thực thi: " + joinPoint.getSignature().getName());
    }

    @After("execution(* com.example.service.OrderService.*(..))")
    public void logAfterMethodCall(JoinPoint joinPoint) {
        System.out.println("LOG: Đã thực thi xong: " + joinPoint.getSignature().getName());
    }
}
```

### Câu hỏi 02: Pointcut, Join Point, Advice, Aspect, Weaving là gì?

Đây là những thuật ngữ cốt lõi của AOP:

* **Join Point**: Một điểm trong quá trình thực thi của một chương trình mà tại đó hành vi có thể bị thay đổi bởi AOP. **Trong Spring AOP, một join point luôn là việc thực thi một phương thức.**
* **Pointcut**: Một biểu thức (predicate) được sử dụng để khớp (match) với các join point. Nó xác định "ở đâu" trong mã nguồn mà advice sẽ được áp dụng.
* **Advice**: Hành vi bổ sung sẽ được chèn vào mã nguồn tại mỗi join point được khớp bởi pointcut. Đây là "cái gì" và "khi nào" sẽ được thực thi.
* **Aspect**: Một module tập hợp `Pointcut` và `Advice` lại với nhau. Nó thường đại diện cho một mối quan tâm xuyên suốt duy nhất (ví dụ: một aspect cho việc quản lý giao dịch).
* **Weaving**: Quá trình áp dụng các aspect vào mã nguồn, sửa đổi hành vi của mã tại các join point đã được khớp. Spring AOP thực hiện **Runtime Weaving**, nghĩa là các aspect được "dệt" vào lúc ứng dụng đang chạy.

### Câu hỏi 03: Spring giải quyết (triển khai) một mối quan tâm xuyên suốt như thế nào?

Spring triển khai các mối quan tâm xuyên suốt bằng cách sử dụng module **Spring AOP**. Spring AOP sử dụng **Runtime Weaving**. Đối với mỗi bean cần được áp dụng aspect, Spring tạo ra một **đối tượng proxy** để chặn các lời gọi phương thức. Có hai loại proxy:

1.  **JDK Dynamic Proxy**: Được tạo ra theo mặc định nếu lớp mục tiêu implement một interface.
2.  **CGLIB Proxy**: Được tạo ra nếu lớp mục tiêu không implement bất kỳ interface nào. Bạn cũng có thể buộc Spring sử dụng CGLIB bằng cách thiết lập `@EnableAspectJAutoProxy(proxyTargetClass = true)`.

Khi client gọi một phương thức trên bean, nó thực sự đang gọi proxy. Proxy này sẽ thực thi logic của `Advice` (ví dụ: ghi log trước), sau đó ủy quyền lời gọi đến phương thức gốc trên đối tượng thật, và cuối cùng có thể thực thi thêm logic của `Advice` (ví dụ: ghi log sau).

### Câu hỏi 04: Những hạn chế của hai loại proxy là gì?

Cả hai loại proxy đều có những hạn chế:
**Hạn chế của JDK Dynamic Proxy:**

* **Phải implement interface**: Lớp mục tiêu bắt buộc phải implement ít nhất một interface.
* **Chỉ các phương thức của interface được proxy**: Chỉ các lời gọi đến các phương thức được định nghĩa trong interface mới bị chặn lại. Các phương thức khác trong lớp sẽ không được áp dụng AOP.
* **Không hỗ trợ tự gọi (self-invocation)**: Nếu một phương thức bên trong đối tượng mục tiêu gọi một phương thức khác của chính nó, lời gọi này sẽ không đi qua proxy và do đó advice sẽ không được áp dụng.

**Hạn chế của CGLIB Proxy:**

* **Không thể proxy lớp/phương thức `final`**: CGLIB hoạt động bằng cách tạo một lớp con của lớp mục tiêu. Do đó, nó không thể proxy một lớp được đánh dấu là `final` hoặc các phương thức `final`.
* **Không hỗ trợ tự gọi (self-invocation)**: Tương tự như JDK Proxy.
* Các phương thức `private` sẽ không được proxy.

### Câu hỏi 05: Spring hỗ trợ bao nhiêu loại advice?

Spring hỗ trợ 5 loại advice chính, mỗi loại được đại diện bởi một annotation:

1.  `@Before`: Thực thi **trước khi** join point được thực thi. Thường dùng cho việc kiểm tra bảo mật, ghi log bắt đầu.
2.  `@After`: Thực thi **sau khi** join point kết thúc, bất kể nó kết thúc bình thường hay ném ra một ngoại lệ. Thường dùng cho việc dọn dẹp tài nguyên.
3.  `@AfterReturning`: Thực thi chỉ khi join point thực thi **thành công** và trả về một kết quả. Có thể truy cập vào giá trị trả về.
4.  `@AfterThrowing`: Thực thi chỉ khi join point **ném ra một ngoại lệ**. Có thể truy cập vào ngoại lệ đó. Dùng để xử lý lỗi hoặc ghi log lỗi.
5.  `@Around`: Đây là advice mạnh mẽ nhất. Nó "bao bọc" xung quanh join point, cho phép bạn kiểm soát hoàn toàn việc thực thi. Bạn có thể thực hiện logic trước, sau, quyết định có gọi phương thức gốc hay không (bằng cách gọi `ProceedingJoinPoint.proceed()`), thay đổi các đối số hoặc giá trị trả về. Thường dùng cho quản lý giao dịch, caching, đo lường hiệu suất.

**Để bắt và xử lý ngoại lệ**, bạn có thể sử dụng hai loại advice: `@AfterThrowing` và `@Around` (bằng cách đặt `proceed()` trong một khối `try-catch`).

#### Ví dụ Code: Các loại Advice

```java
@Aspect
@Component
public class ComprehensiveAspect {

    // Thực thi trước mỗi phương thức trong package service
    @Before("execution(* com.example.service.*.*(..))")
    public void doBefore(JoinPoint joinPoint) {
        System.out.println("ADVICE: @Before - Chuẩn bị thực thi: " + joinPoint.getSignature().getName());
    }

    // Thực thi sau khi phương thức trả về thành công
    @AfterReturning(
        pointcut="execution(* com.example.service.DataService.fetchData(..))", 
        returning="result"
    )
    public void doAfterReturning(Object result) {
        System.out.println("ADVICE: @AfterReturning - Phương thức trả về: " + result);
    }

    // Thực thi khi có lỗi xảy ra
    @AfterThrowing(
        pointcut="execution(* com.example.service.DataService.processWithError(..))", 
        throwing="error"
    )
    public void doAfterThrowing(Throwable error) {
        System.out.println("ADVICE: @AfterThrowing - Đã xảy ra lỗi: " + error.getMessage());
    }

    // Thực thi xung quanh một phương thức để đo thời gian
    @Around("execution(* com.example.service.PerformanceService.runTask(..))")
    public Object profile(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("ADVICE: @Around - Bắt đầu đo thời gian.");

        Object output = pjp.proceed(); // Thực thi phương thức gốc

        long elapsedTime = System.currentTimeMillis() - start;
        System.out.println("ADVICE: @Around - Thời gian thực thi: " + elapsedTime + "ms.");
        return output;
    }
}
```

### Câu hỏi 06: Bạn phải làm gì để kích hoạt việc phát hiện annotation @Aspect?

Để Spring phát hiện và xử lý các lớp được chú thích bằng `@Aspect`, bạn cần:

1.  **Thêm annotation `@EnableAspectJAutoProxy`** vào một trong các lớp `@Configuration` của bạn. Annotation này sẽ kích hoạt việc tạo proxy cho các bean phù hợp với các aspect.
2.  **Đảm bảo các lớp Aspect là Spring bean**: Lớp được chú thích `@Aspect` cũng phải được khai báo là một bean để Spring Container quản lý nó. Cách phổ biến nhất là thêm một annotation stereotype như `@Component` vào lớp aspect và đảm bảo component-scanning được bật.
3.  **Thêm các dependency cần thiết**: Đảm bảo rằng bạn có `spring-aop` và `aspectjweaver` trên classpath.

#### Ví dụ Code: Cấu hình

```java
@Configuration
@ComponentScan("com.example") // Đảm bảo các @Component và @Aspect được quét
@EnableAspectJAutoProxy       // Kích hoạt hỗ trợ AOP
public class AppConfig {
    // ... các định nghĩa @Bean khác nếu có
}
```

### Câu hỏi 07: Bạn có hiểu các biểu thức pointcut không?

Biểu thức pointcut là một phần cốt lõi của AOP, cho phép bạn chỉ định chính xác nơi áp dụng advice. Spring sử dụng ngôn ngữ biểu thức của AspectJ. Một số designator phổ biến bao gồm:

* `execution()`: Khớp với việc thực thi một phương thức. Đây là designator mạnh mẽ và được sử dụng nhiều nhất.
* `within()`: Khớp với tất cả các join point bên trong một kiểu (class) hoặc package nhất định.
* `@annotation()`: Khớp với các join point có annotation được chỉ định.
* `bean()`: Khớp với các phương thức của một Spring bean có tên cụ thể.
* `args()`: Khớp với các join point có các đối số phù hợp.
* `this()` và `target()`: Khớp dựa trên kiểu của đối tượng proxy (`this`) hoặc đối tượng mục tiêu (`target`).

Bạn có thể kết hợp các biểu thức này bằng các toán tử logic `&&` (và), `||` (hoặc), `!` (phủ định).

**Ví dụ, biểu thức pointcut để khớp với cả phương thức getter và setter trong lớp `EmployeeBean`:**

```java
execution(* com.beans.EmployeeBean.get*()) || execution(* com.beans.EmployeeBean.set*(*))
```

### Câu hỏi 08: Đối số JoinPoint được dùng để làm gì?

Đối số `JoinPoint` là một đối tượng mà Spring có thể tiêm vào một phương thức advice (ngoại trừ `@Around`). Nó được sử dụng để truy xuất thông tin bổ sung (metadata) về join point đang được thực thi.

Bạn có thể lấy được:

* `getArgs()`: Các đối số của phương thức.
* `getSignature()`: Chữ ký của phương thức (tên, kiểu trả về, tham số).
* `getTarget()`: Đối tượng mục tiêu (đối tượng gốc).
* `getThis()`: Đối tượng proxy.

#### Ví dụ Code

```java
@Before("execution(* com.example.service.UserService.updateUser(..))")
public void inspectMethodCall(JoinPoint joinPoint) {
    System.out.println("Phương thức được gọi: " + joinPoint.getSignature().toShortString());
    Object[] args = joinPoint.getArgs();
    System.out.println("Với các đối số: " + Arrays.toString(args));
}
```

### Câu hỏi 09: ProceedingJoinPoint là gì? Khi nào nó được sử dụng?

`ProceedingJoinPoint` là một lớp con của `JoinPoint`, và nó chỉ được sử dụng làm đối số cho các advice kiểu `@Around`.

Nó kế thừa tất cả các phương thức của `JoinPoint` nhưng thêm một phương thức cực kỳ quan trọng:

* `proceed()`: Thực thi join point tiếp theo trong chuỗi (thường là phương thức gốc).
* `proceed(Object[] args)`: Thực thi join point với một mảng các đối số mới.

Vì `@Around` advice "bao bọc" phương thức gốc, nó có trách nhiệm quyết định có nên tiếp tục chuỗi thực thi hay không bằng cách gọi `proceed()`. Nếu `proceed()` không được gọi, phương thức gốc sẽ không bao giờ được thực thi. Điều này cho phép bạn chặn hoàn toàn việc thực thi một phương thức, thay đổi đối số của nó, hoặc xử lý giá trị trả về của nó.