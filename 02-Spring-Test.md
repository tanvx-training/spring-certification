### Spring Test

- **Unit Testing**
    - Kiểm thử một đơn vị chức năng
    - Giữ phụ thuộc ở mức tối thiểu
    - Cô lập khỏi môi trường (bao gồm cả Spring)
- **Integration Testing**
    - Kiểm thử sự tương tác của nhiều đơn vị làm việc cùng nhau
    - Tích hợp cơ sở hạ tầng
- `TDD` - Phương pháp này đặt câu hỏi về thiết kế: nếu việc viết kiểm thử gặp khó khăn, thiết kế cần được xem xét lại
- Cố gắng viết ít nhất hai kiểm thử cho mỗi phương thức: một kiểm thử dương tính và một kiểm thử âm tính, đối với những phương thức có thể kiểm thử

- Để định nghĩa một lớp kiểm thử trong bối cảnh Spring, cần làm những điều sau:
    - Thêm `@RunWith(SpringJUnit4ClassRunner.class)` vào lớp kiểm thử
    - Thêm `@ContextConfiguration` vào lớp để chỉ định vị trí của các bean cần cấu hình
    - Sử dụng `@Autowired` để tiêm các bean vào kiểm thử.

- Phân phối dưới dạng artifact riêng biệt - `spring-test.jar`
- Lớp kiểm thử cần được chú thích với `@RunWith(SpringJUnit4ClassRunner.class)`
- Các lớp cấu hình Spring cần được tải được chỉ định trong `@ContextConfiguration`
    - `@ContextConfiguration` xác định siêu dữ liệu lớp dùng để xác định cách tải và cấu hình `ApplicationContext` cho kiểm thử tích hợp.
      Cụ thể, `@ContextConfiguration` khai báo các tài nguyên **locations** hoặc **classes** của ứng dụng mà sẽ được sử dụng để tải context.
    - Nếu không cung cấp giá trị cho `@ContextConfiguration`, file cấu hình `${classname}-context.xml` trong cùng package sẽ được nhập vào
    - **File cấu hình XML** được tải bằng cách cung cấp giá trị chuỗi vào anotations - `@ContextConfiguration("classpath:com/example/test-config.xml")`
    - **File Java `@Configuration`** được tải từ thuộc tính classes - `@ContextConfiguration(classes={TestConfig.class, OtherConfig.class})`

- Để tùy chỉnh giá trị thuộc tính trong kiểm thử, anotations `@TestPropertySource` cho phép sử dụng file thuộc tính riêng hoặc tùy chỉnh từng giá trị thuộc tính.
- `ReflectionTestUtils` giúp truy cập các thuộc tính private:
    - Thay đổi giá trị của các hằng số
    - Gán giá trị hoặc tham chiếu vào trường không công khai.
    - Gọi phương thức setter không công khai.
    - Gọi phương thức callback không công khai.
- Spring có các đối tượng giả lập trên `Environment`, `JNDI`, và `Servlet API` để hỗ trợ kiểm thử đơn vị.
- **Mặc định, framework sẽ tạo và cuộn lại một giao dịch cho mỗi kiểm thử.**
- Các kiểm thử được chú thích với `@Transactional` nhưng có kiểu propagation là `NOT_SUPPORTED` sẽ không được chạy trong giao dịch.

#### Tạo ApplicationContext trong kiểm thử tích hợp

Tùy thuộc vào việc sử dụng JUnit 4 hoặc JUnit 5, anotations `@RunWith` (JUnit 4) hoặc `@ExtendWith` (JUnit 5) sẽ được sử dụng để chú thích lớp kiểm thử. Ngoài ra, anotations `@ContextConfiguration` cũng được sử dụng để chỉ định file cấu hình XML hoặc lớp Java chứa cấu hình Spring cần tải vào ApplicationContext cho kiểm thử.

#### Ví dụ JUnit 4

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes={TestConfig.class, OtherConfig.class})
public final class FooTest  {
 
    @Autowired
    private MyService myService;
    
    @Test
    public void test() {
    	
    }
}
```

hoặc

- `AbstractJUnit4SpringContextTests` triển khai `ApplicationContextAware` và do đó cung cấp quyền truy cập tự động vào `ApplicationContext`.

```java
@ContextConfiguration
public class MyTestClass extends AbstractJUnit4SpringContextTests {

}
```

#### Ví dụ JUnit 5

```java
@SpringJUnitConfig(classes={TestConfig.class, OtherConfig.class})
public final class FooTest  {
 
    @Autowired
    private MyService myService;
    
    @Test
    public void test() {
    	
    }
}
```

#### Kiểm thử Web Application Context

- Kiểm thử đơn vị Spring với `@WebAppConfiguration`
    - Tạo `WebApplicationContext`
    - Có thể kiểm thử mã sử dụng các tính năng web
        - ServletContext, phạm vi bean Session và Request
- Định cấu hình vị trí tài nguyên
    - Mặc định là `src/main/webapp`
    - Đối với tài nguyên classpath sử dụng tiền tố `classpath:`

```java
@SpringJUnitConfig(classes={TestConfig.class, OtherConfig.class})
@WebAppConfiguration
public final class FooTest  {
 
    @Autowired
    private MyService myService;
    
    @Autowired
    private WebApplicationContext mWebApplicationContext;
    
    @Test
    public void test() {
    	
    }
}
```

- `ApplicationContext` chỉ được **khởi tạo một lần** cho tất cả các kiểm thử sử dụng cùng một tập tin cấu hình (ngay cả giữa các lớp kiểm thử)
- Thêm anotations `@DirtiesContext` vào phương thức kiểm thử để yêu cầu tái tạo lại `ApplicationContext` nếu phương thức thay đổi các bean bên trong.

#### AnnotationConfigContextLoader

- Các lớp `@Configuration` (phải là lớp static) tự động được phát hiện và tải trong kiểm thử với sự trợ giúp của `AnnotationConfigContextLoader`

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(loader = AnnotationConfigContextLoader.class)
public class SpringPetServiceTest3 {

     @Configuration
     public static class TestCtxConfig {
        @Bean
        StubPetRepo petRepo(){
            return new StubPetRepo();
        }
        @Bean
        PetService simplePetService(){
            SimplePetService petService = new SimplePetService();
            petService.setRepo(petRepo());
            return petService;
        }
    }
}
```

#### Kiểm thử với Spring Profiles

- Anotations `@ActiveProfiles` trong lớp kiểm thử kích hoạt các profile đã liệt kê
- `@ActiveProfiles( { "foo", "bar" } )`

#### Tiêm Mock bằng Mockito

- Anotations `@InjectMock` có hành vi tương tự Spring IoC, vì nó có vai trò tạo các đối tượng kiểm thử và cố gắng tiêm các trường được anotations `@Mock` hoặc `@Spy` vào các trường private của đối tượng kiểm thử.

```java
public class MockPetServiceTest {
    
    @InjectMocks
    private SimplePetService simplePetService;
    
    @Mock
    private PetRepo petRepo;
    
    @Before
    public void initMocks() {
        MockitoAnnotations.initMocks(this);
    }
}
```

- Với `@RunWith(MockitoJUnitRunner.class)` không cần gọi `MockitoAnnotations.initMocks(this)`

```java
@RunWith(MockitoJUnitRunner.class)
public class MockPetServiceTest {

    @InjectMocks
    private SimplePetService simplePetService;

    @Mock
    private PetRepo petRepo;

}
```

- `PowerMock` ra đời vì đôi khi mã không thể kiểm thử được, có thể vì thiết kế không tốt hoặc do một số yêu cầu. Dưới đây là danh sách các phần không thể kiểm thử:
    - Các phương thức tĩnh
    - Các lớp có static initializers
    - Các lớp final và phương thức final
    - Các phương thức và trường private

### Testing Rest với Spring Boot

- Có thể sử dụng `SpringRunner` thay thế cho `SpringJUnit4ClassRunner`

```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = CustomerController.class, secure=false)
public class TestCustomerController {

	@MockBean
	private CustomerService service;

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void testSuccessfulFindAllCustomers() throws Exception {
		when(service.findAllCustomers()).thenReturn(Arrays.asList(new Customer(), new Customer()));

		mockMvc.perform(get("/customers"))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
            .andExpect(jsonPath("$", hasSize(2)));
	}
}
```

```java
@RestController
@RequestMapping("/customers")
public class CustomerController {

	private CustomerService service;

	public CustomerController(CustomerService service) {
		this.service = service;
	}
	
	@GetMapping
	public ResponseEntity<Iterable<Customer>> findAllCustomers(){
		return ResponseEntity.ok(service.findAllCustomers());
	}
}   
```

### Kiểm thử JPA với Spring Boot

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class TestCustomerRepo {

	@Autowired
	private TestEntityManager entityManager;

	@Autowired
	private CustomerRepo repo;
	
	private Customer bojack;

	public TestCustomerRepo() {
		bojack = new Customer.CustomerBuilder().firstName("BoJack").middleName("Horse").lastName("Horseman")
				.suffix("Sr.").build();
	}

	@Test
	public void testFindAllCustomers() {
		this.entityManager.persist(bojack);
		Iterable<Customer> customers = repo.findAll();

		int count = 0;
		for (Customer repoCustomer : customers) {
			assertEquals("BoJack", repoCustomer.getFirstName());
			assertEquals("Horseman", repoCustomer.getLastName());
			assertEquals("Horse", repoCustomer.getMiddleName());


			assertEquals("Sr.", repoCustomer.getSuffix());
			count++;
		}
		assertEquals(1, count);
	}
}
```