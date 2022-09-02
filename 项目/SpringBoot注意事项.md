## 前后端ID不一致

后端返回Long型的id数据时，超过了前端js的最大范围，会不一致。因此需要返回String类型id，或者在Vo类属性设置注解

```java
@JsonSerialize(using = ToStringSerializer.class)
private Long id;
```

## 实体类属性应设成包装类，如Integer等

因为执行更新操作时，要创建一个只包含一个要更新属性的实体对象，若是使用基本类型，则默认为0，会将其他属性设置成0

## 关于空指针异常

测试类没有添加 @SpringBootTest

Bean对象没有添加 @Autowired

## ”实例化“接口对象

实质上是因为接口实现类添加了 @Service 注解，使用 @Autowired 时，造成可以直接实例化接口对象的假象

```java
public interface BookService {
    Boolean save(Book book);

    Boolean update(Book book);

    Boolean delete(Integer id);

    Book getById(Integer id);

    List<Book> getAll();

    IPage<Book> getPage(int currentPage, int pageSize);
}
```

```java
@Service
public class BookServiceImpl implements BookService {

    @Autowired
    BookDao bookDao;

    @Override
    public Boolean save(Book book) {
        return bookDao.insert(book) > 0;
    }

    @Override
    public Boolean update(Book book) {
        return bookDao.updateById(book) > 0;
    }

    @Override
    public Boolean delete(Integer id) {
        return bookDao.deleteById(id) > 0;
    }

    @Override
    public Book getById(Integer id) {
        return bookDao.selectById(id);
    }

    @Override
    public List<Book> getAll() {
        return bookDao.selectList(null);
    }

    @Override
    public IPage<Book> getPage(int currentPage, int pageSize) {
        IPage<Book> page = new Page<>(currentPage, pageSize);
        bookDao.selectPage(page, null);
        return page;
    }
}
```

```java
@Autowired
private BookService bookService;
```

## SpringBoot测试类bean无法加载

解决：测试类的包名要和主目录包名一模一样

https://blog.csdn.net/m0_62436868/article/details/124842389

![image-20220617200507884](C:\Users\98449\AppData\Roaming\Typora\typora-user-images\image-20220617200507884.png)

![image-20220617200537817](C:\Users\98449\AppData\Roaming\Typora\typora-user-images\image-20220617200537817.png)

