title: 使用Mockito和SpringTest进行单元测试
keywords: 'Mockito,SpringTest,单元测试'
date: 2017-11-13 21:03:04
categories: Java 
tags: 单元测试
description:
---
<blockquote class="blockquote-center">
单元测试的作用以及重要性其实不用在这里不断的重复了.为了保证软件的可靠性,单元测试可能是最容易执行的一个可靠的手段了.一方面，程序员通过编写单元测试来验证自己程序的有效性,另外一方面,管理者通过持续自动的执行单元测试和分析单元测试的覆盖率等来确保软件本身的质量.
</blockquote>



<!--more-->
JAVA生态圈里面说起单元测试一般都会使用JUnit或者TestNG.其中JUnit可能使用的更加频繁一些,JUnit4使用注解以及各种其他框架对它的支持,形成了一个完善的单元测试的生态圈.
##SpringTest单元测试
现在的JAVA WEB项目中,起码一半以上的项目是使用了Spring的.因此,单纯的使用JUnit来进行单元测试并不是十分的好用,对于由Spring管理的Bean要进行单元测试,首先需要实例化Spring上下文,然后又需要手动的去注入依赖的Bean,比较麻烦.特别是对于有事务的单元测试,或数据库数据测试,单独使用JUnit几乎无法完成.
所幸,SpringFramework也意识到了这点,于是推出了spring-test模块,他能完成Spring环境与Junit单元测试环境的对接.让我们只专注于单元测试本身进行书写,而由它来完成Spring容器的初始化、Bean的获取、数据库事务的管理、数据操作正确性检查等等。
项目中引用spring-test
要在项目中使用Spring测试框架非常的简单，在Maven中依赖spring-test即可，Maven会自动的完成依赖包的引用：
  ```xml
  <dependencies>
<!--test-->
<dependency>
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>4.12</version>
<scope>test</scope>
</dependency>
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-test</artifactId>
<version>4.2.0.RELEASE</version>
<scope>test</scope>
</dependency>
</dependencies>
  ```
## 创建单元测试类
Spring-test以及Junit4都推荐使用注解的方式来进行配置。因此，我们要进行单元测试，需要的就是创建一个自己的测试类。然后在类上面进行少许的注解，表明我们要如何初始化spring，需要引用spring的哪些配置，然后再指明需要在单元测试类中注入哪些bean即可，比如：
  ```java
  @RunWith(SpringJUnit4ClassRunner.class) // 整合Spring-Test与JUnit
@ContextConfiguration(locations = {"classpath:application_bean_commons.xml",
"classpath:application_bean_rmdbaccess.xml",
"classpath:application_pm_service_poolmanage.xml"}) // 加载需要的配置配置
@Transactional //事务回滚,便于重复测试
public class TestPoolService{

@Autowired
private IPoolService poolService;

@Before
public void setUp() {
         //可以在每一次单元测试前准备执行一些东西
}

@Test
public void testAddPool() {

Pool pool = new Pool();
pool.setPoolName("test_pool_001");
pool.setDescription("test add pool");
pool = poolService.addPool(pool);

//进行结果的验证
assertNotNull(pool.getId());
assertNotNull(pool.getPoolCode());
}
}
  ```
  从这个简单的例子当中，我们可以看到和单独使用JUnit有几个不一样的地方。

	1. 首先使用了@RunWith(SpringJUnit4ClassRunner.class)来告诉JUnit，需要结合Spring-test来执行单元测试
	2. 然后使用@ContextConfiguration注解来告诉Spring-test这个单元测试最小需要依赖的spring的上下文是什么，因为通常单元测试不需要引入所有的spring配置，因此我们可以在这里可选的读取几个需要的配置即可
	3. 而后，我们可以在类上或者方法上使用@Transactional注解来标识某个或某些单元测试用例需要使用事务，并且可能会进行回滚，防止单元测试引起数据库脏数据。
	4. 同时，我们可以在这个单元测试类中直接使用@Autowired以及@Qualifier注解直接注入经过Spring管理过的依赖Bean。而我们测试的主要目的也就是对这些bean进行单元测试。
	5. 剩下的就和单独的使用JUnit进行单元测试是一样的了。

经过以上的处理，我们就可以进行spring框架下的单元测试了，而且基本上能满足80%以上的需求。更多更高级的用法，可以参考：[JUnit官网][1]以及[Spring-test官网][2]

##基于数据库数据的单元测试数据准备
其实这个和要讲的Spring-test没有必然的联系。但是在很多的情况下，我们都会对数据库访问层进行单元测试。那么，往往就涉及到了数据库的数据打桩。为了能实现单元测试的自动化和可重复化，我们可以把桩数据写入一个单元测试的SQL中，在每次执行单元测试的时候，先执行这个SQL，给数据库准备数据，然后再执行单元测试。而这一切，我们可以写一个测试的基类来进行处理（JDK1.8中允许给接口增加默认方法，因此，这个地方我们可以定义一个接口来实现这个功能）：
  ```java
  public interface DBTestBase {

// 接口的默认方法，对数据库进行打桩
default void prepareRmdbData(SimpleRmdbDao simpleRmdbDao) {

String content = sqlForThisTest();

if (content==null){
return;
}
String[] sqlLines = content.split(";");

for (String sqlLine : sqlLines) {
simpleRmdbDao.executeDDL(sqlLine);
}
}

// 解析和这个测试类相同名字的SQL，一行一句。
default String sqlForThisTest() {
String sqlName = getClass().getSimpleName() + ".sql";

try {
List<String> lines = IOUtils.readLines(getClass().getResourceAsStream(sqlName),"utf-8");

/*去掉SQL中的注释*/
Optional<String> optional = lines.parallelStream().filter(s -> !(s.startsWith("-- ")&&s.endsWith(" --")))
.reduce(String::concat);

if (optional.isPresent()){
return optional.get();
}
return null;
} catch (IOException e) {
e.printStackTrace();
}
return null;
}
}

  ```
  
有了这个接口，我们在写单元测试类的时候，就可以直接实现这个接口，然后在@Before方法中调用prepareRmdbData方法来初始化数据库:
  ```java
  @RunWith(SpringJUnit4ClassRunner.class) // 整合Spring-Test与JUnit
@ContextConfiguration(locations = {"classpath:application_bean_commons.xml",
"classpath:application_bean_rmdbaccess.xml",
"classpath:application_pm_service_poolmanage.xml"}) // 加载需要的配置配置
@Transactional //事务回滚,便于重复测试
public class TestPoolService implements DBTestBase {

@Autowired
private IPoolService poolService;

@Autowired
private SimpleRmdbDao simpleRmdbDao;

@Before
public void setUp() {
         //可以在每一次单元测试前准备执行一些东西
         prepareRmdbData(simpleRmdbDao);
}

@Test
public void testDeletePool() {

//case1
Pool pool = poolService.getPoolByCode("P-00003");

assertNotNull(pool);

poolService.deletePool("P-00003");

pool = poolService.getPoolByCode("P-00003");

assertNull(pool);
}
}

  ```

然后把SQL文件命名为TestPoolService.sql,放入test/resources中即可。
Mock测试
经过上面所说的JUnit+SpringTest,基本上可以满足80%的单元测试了。但是，由于现在的系统越来越复杂，相互之间的依赖越来越多。特别是微服务化以后的系统，往往一个模块的代码需要依赖几个其他模块的东西。因此，在做单元测试的时候，往往很难构造出需要的依赖。一个单元测试，我们只关心一个小的功能，但是为了这个小的功能能跑起来，可能需要依赖一堆其他的东西，这就导致了单元测试无法进行。所以，我们就需要再测试过程中引入Mock测试。
所谓的Mock测试就是在测试过程中，对于一些不容易构造的、或者和这次单元测试无关但是上下文又有依赖的对象，用一个虚拟的对象（Mock对象）来模拟，以便单元测试能够进行。
比如有一段代码的依赖为：
![此处输入图片的描述][3]
当我们要进行单元测试的时候，就需要给A注入B和C,但是C又依赖了D，D又依赖了E。这就导致了，A的单元测试很难得进行。
但是，当我们使用了Mock来进行模拟对象后，我们就可以把这种依赖解耦，只关心A本身的测试，它所依赖的B和C，全部使用Mock出来的对象，并且给MockB和MockC指定一个明确的行为。就像这样：

![此处输入图片的描述][4]
因此，当我们使用Mock后，对于那些难以构建的对象，就变成了个模拟对象，只需要提前的做Stubbing（桩）即可，所谓做桩数据，也就是告诉Mock对象，当与之交互时执行何种行为过程。比如当调用B对象的b()方法时，我们期望返回一个true，这就是一个设置桩数据的预期。

##Mockito简单入门
在JAVA中，Mock测试框架主要有Mockito,Jmock,EsayMock,PowerMock等等。其中Mockito最为方便和简单，用的人也最多。而PowerMock是对Mockito的一个增强,增加了对静态、final、私有方法的Mock，但是基本用法和Mockito大致相同。对因此，我们使用Mockito作为Mock的框架。
项目中引用Mockito
要在项目中使用Mockito非常的简单，只需要在项目的Maven中引入：
  ```xml
  <dependency>
<groupId>org.mockito</groupId>
<artifactId>mockito-core</artifactId>
<version>2.1.0</version>
<scope>test</scope>
</dependency>
  ```
###使用Mockito进行测试
我们这里使用一个最简单的用户基本信息管理来做演示。这个功能有一个模型对象UserPO,一个数据库访问层UserDao,一个服务层UserService。
*UserPO*
  ```java
  public class UserPO implements Serializable {

private Long id;

private String name;

private Integer age;

public Long getId() {
return id;
}

public void setId(Long id) {
this.id = id;
}

public String getName() {
return name;
}

public void setName(String name) {
this.name = name;
}

public Integer getAge() {
return age;
}

public void setAge(Integer age) {
this.age = age;
}
}

  ```
*UserDao*
  ```java
  public interface IUserDao {

public Boolean updateUser(UserPO userPO);

public UserPO getUserById(Long id);

}

  ```
*UserService*  
  ```java
  public class UserService {

private IUserDao userDao;

public void setUserDao(IUserDao userDao) {
this.userDao = userDao;
}

public boolean updateUserName(Long userId,String name){

UserPO userPO = userDao.getUserById(userId);

if (userPO==null){
return false;
}

userPO.setName(name);

return userDao.updateUser(userPO);
}
}

  ```
  
###创建单元测试类
当我们准备好上面的例子后，就可以开始创建单元测试类了。
在这里，我们假设IUserDao是个很复杂的访问集群数据库的对象，并且这个类已经经过完整的测试保证是正确了的。而我们现在只需要单元测试UserService#updateUserName这个方法。因此，就需要对IUserDao进行Mock并打桩。
  ```java
  public class UserServiceTest {

private UserService userService;

@Mock
private IUserDao userDao;

@Before
public void setUp() {
//对注解了@Mock的对象进行模拟
MockitoAnnotations.initMocks(this);

//构造被测试对象
userService = new UserService();
userService.setUserDao(userDao);

//数据打桩。 当调用 userDao.getUserById(1L)时，返回一个UserPO
Mockito.when(userDao.getUserById(1L)).thenReturn(new UserPO(1L,"user1",20));
// 当调用 userDao.getUserById(2L)时，返回一个null，表示用户不存在
Mockito.when(userDao.getUserById(2L)).thenReturn(null);

// 当调用userDao.updateUser(userPO)的时候，返回一个true
Mockito.when(userDao.updateUser(Mockito.any())).thenReturn(true);

}

@Test
public void testUpdateUserNameSuccess() {
/*成功的情况*/

/*测试这个被测试对象的方法*/
boolean updated = userService.updateUserName(1L,"user_new");

//验证结果
Assert.assertTrue(updated);

//验证userDao的getUserById(1L)这个方法是否被调用过
Mockito.verify(userDao).getUserById(1L);

//构造参数捕获器，用于捕获方法参数进行验证
ArgumentCaptor<UserPO> personCaptor = ArgumentCaptor.forClass( UserPO.class );
//验证updateUser方法是否被调用过，并且捕获入参
Mockito.verify(userDao).updateUser(personCaptor.capture());

//返回捕获的参数。
UserPO updatedPerson = personCaptor.getValue();
//判断是否已经被修改过了
Assert.assertEquals("user_new", updatedPerson.getName());

//多余方法调用验证，保证这个测试用例中所有被Mock的对象的相关方法都已经被Verify过了
Mockito.verifyNoMoreInteractions(userDao);

}

@Test
public void testUpdateUserNameFailed() {
/*测试这个被测试对象的方法*/
boolean updated = userService.updateUserName(2L,"user_new");

//验证结果
Assert.assertFalse(updated);

//验证userDao的getUserById(1L)这个方法是否被调用过
Mockito.verify(userDao).getUserById(2L);
//多余方法调用验证，保证这个测试用例中所有被Mock的对象的相关方法都已经被Verify过了
Mockito.verifyZeroInteractions(userDao);
//多余方法调用验证，保证这个测试用例中所有被Mock的对象的相关方法都已经被Verify过了
Mockito.verifyNoMoreInteractions(userDao);
}
}

  ```
  上面的代码就是一个完整的单元测试类，有两个用例，分别验证当用户存在能修改名字的情况以及用户不存在修改名字失败的情况。
我们从头来分析一下这个单元测试类。
###标明需要Mock的对象
程序一来，先定义了被测试的对象实例userService以及需要被模拟的IUserDao对象。需要注意的是，我们在userDao成员变量上增加了一个@Mock注解。这个注解的作用就是告诉Mockito，这个对象是需要被Mock的。
接着，我们创建了一个setUp()方法，并使用了JUnit的注解@Before，用于在执行单元测试前执行一些代码，我们在这里需要对Mock的对象进行打桩。
MockitoAnnotations.initMocks(this);这句话就是对所有标注了@Mock注解的对象进行模拟。当然，我们也可以不使用注解，而直接使用代码的方式手动的初始化Mock的对象：
  ```java
  //对注解了@Mock的对象进行模拟
// MockitoAnnotations.initMocks(this);

//使用手动的方式进行Mock
userDao = Mockito.mock(IUserDao.class);
  ```
  接着就是指定userDao的行为也就是桩了。这也是Mockito最常用最核心的方法了。
  ```java
  //数据打桩。 当调用 userDao.getUserById(1L)时，返回一个UserPO
Mockito.when(userDao.getUserById(1L)).thenReturn(new UserPO(1L,"user1",20));
// 当调用 userDao.getUserById(2L)时，返回一个null，表示用户不存在
Mockito.when(userDao.getUserById(2L)).thenReturn(null);

// 当调用userDao.updateUser(userPO)的时候，返回一个true
Mockito.when(userDao.updateUser(Mockito.any())).thenReturn(true);

  ```
  Mockito最基本的用法就是调用 when以及thenReturn方法了。他们的作用就是指定当我们调用被代理的对象的某一个方法以及参数的时候，返回什么值。

* 比如第一句的Mockito.when(userDao.getUserById(1L)).thenReturn(new UserPO(1L,"user1",20));就表明，当我调用userDao.getUserById(1L)的时候，这个方法返回new UserPO(1L,"user1",20)这个实例。
* 当我们需要返回NULL的时候，也非常的简单，直接写成thenReturn(null)即可。
* 如果我们不关心调用的参数的入参，那么Mockito提供了几个方法来表示:any()、any(Class<T> type)、anyBoolean()、anyByte()、anyChar()、anyInt()、anyLong()、anyFloat()、anyDouble()、anyShort()、anyString()、anyList()、anyListOf(Class<T> clazz)、anySet()、anyMap()等等
* 相反，Mockito还提供了很强大的入参过滤，用于指定只对某一些入参的调用进行Mock。比如：正则表达式Mockito.matches(".*User$"))、开头结尾验证endsWith(String suffix) startsWith(String prefix)、判空验证isNotNull() isNull()
* 甚至，我们还可以自定义入参匹配：argThat(ArgumentMatcher<T> matcher)。ArgumentMatcher只有一个方法boolean matches(T argument);传入入参，返回一个boolean表示是否匹配。在JDK1.8中，我们可以使用lambda表达式来自定义入参匹配，比如：
  ```java
  Mockito.argThat(argument -> argument instanceof UserPO);
  ```
除了我们期望调用一个方法后返回一个值外，有些时候，我们可能期望他抛出一个异常。这个时候，我们可以调用thenThrow(Throwable... throwables); 用来抛出异常，这个方法有三个重载:
  ```java
		* thenThrow(Throwable... throwables): 直接指定抛出的异常实例
		* thenThrow(Class<? extends Throwable> throwableType): 指定抛出异常的类型，执行的时候动态的实例化一个异常实例
		* thenThrow(Class<? extends Throwable> toBeThrown, Class<? extends Throwable>... nextToBeThrown): 多次调用，依次抛出异常
  ```
//当调用userDao的更新时，如果传入的用户的名字是admin，那么就不允许修改，直接抛出异常
  ```java
Mockito.when(userDao.updateUser(Mockito.argThat(argument -> argument.getName().equals("admin")))).thenThrow(IllegalArgumentException.class);
  ```
* 此外，Mockito还提供了两个表示行为的方法：thenAnswer(Answer<?> answer);、thenCallRealMethod();,分别表示自定义处理调用后的行为，以及调用真实的方法。这两个方法在有些测试用例中还是很有用的。
* 对于同一个方法，Mockito可以是顺序与次数关心的。也就是说可以实现同一个方法，第一次调用返回一个值，第二次调用返回一个值，甚至第三次调用抛出异常等等。只需要连续的调用thenXXXX即可。
* 最后，还有一个需要说明的就是如果为一个返回为Void的方法设置桩数据。上面的方法都是表示的是有返回值的方法，而由于一个方法没有返回值，因此我们不能调用when方法(编译器不允许)。因此，对于无返回值的方法，Mockito提供了一些列的doXXXXX方法，比如：doAnswer(Answer answer)、doNothing()、doReturn(Object toBeReturned)、doThrow(Class<? extends Throwable> toBeThrown)、doCallRealMethod()。他们的使用方法其实和上面的thenXXXX是一样的，但是when方法传入的是Mock的对象：
  ```java
  /*对void的方法设置模拟*/
Mockito.doAnswer(invocationOnMock -> {
System.out.println("进入了Mock");
return null;
}).when(fileRecordDao).insert(Mockito.any());

  ```
###验证Mock对象的调用
其实，一个最简单的Mock单元测试到这里已经算是完成了。我们已经验证了userService中的方法的正确性。但是，在复杂的方法调用堆栈中，往往可能出现结果正确，但是过程不正确的情况。比如，updateUserName方法返回false是有两种可能的，一种可能是用户没有找到，还有一种可能就是userDao.updateUser(userPO)返回false。因此，如果我们只是使用Assert.assertFalse(updated);来验证结果，可能就会忽略某些错误。
Mockito同时提供了一些列的方法来对调用过程中的Mock对象的方法调用进行跟踪。我们可以对这些调用的过程进行断言验证，保证单元测试的结果与过程都是符合我们预期的。
  ```java
  @Test
public void testUpdateUserNameSuccess() {
/*成功的情况*/

/*测试这个被测试对象的方法*/
boolean updated = userService.updateUserName(1L,"user_new");

//验证结果
Assert.assertTrue(updated);

//验证userDao的getUserById(1L)这个方法是否被调用过
Mockito.verify(userDao).getUserById(1L);

//构造参数捕获器，用于捕获方法参数进行验证
ArgumentCaptor<UserPO> personCaptor = ArgumentCaptor.forClass( UserPO.class );
//验证updateUser方法是否被调用过，并且捕获入参
Mockito.verify(userDao).updateUser(personCaptor.capture());

//返回捕获的参数。
UserPO updatedPerson = personCaptor.getValue();
//判断是否已经被修改过了
Assert.assertEquals("user_new", updatedPerson.getName());

//多余方法调用验证，保证这个测试用例中所有被Mock的对象的相关方法都已经被Verify过了
Mockito.verifyNoMoreInteractions(userDao);

}

  ```

* Mockito.verify(userDao).getUserById(1L);方法即验证了getUserById(1L)这个方法是否被调用过，如果没有被调用过(包括入参要一致)，就会抛出异常。
* 除了最简单的verify(T mock)方法外，还提供了verify(T mock, VerificationMode mode)方法。第二个参数有很多默认的实现，用于满足不同的需求。比如：Mockito.verify(userDao,Mockito.times(1)).getUserById(1L); 表示调用第一次 是传入的getUserById(1L);，Mockito.verify(userDao,Mockito.times(2)).getUserById(2L);表示调用第二次是传入的getUserById(2L);，如果测试用例的调用顺序与参数不满足的话，就会报错。
* 除了times函数外，还提供了after、atLeast、only、atMost、timeout等等。
* verifyZeroInteractions和verifyNoMoreInteractions这两个方法的实现其实是一样的，只是名字不一样，作用就是验证被Mock的对象的所有被调用的方法是否都被Verify过了。这样就能保证调用没有被遗漏。当有方法被调用了，但是我们在测试用例中没有verify的话，那么调用这两个方法就会抛异常。
##Mockito与SpringTest整合
经过前面的讲解，Mockito的基本用法基本上应该都了解了。那么现在就需要整合Mockito和SpringTest了。
其实这两者的整合也非常的简单。和他们单独使用的时候并没有什么区别。
  ```java
  @RunWith(SpringJUnit4ClassRunner.class) // 整合Spring-Test与JUnit
@ContextConfiguration(locations = {"classpath:application_bean_commons.xml",
"classpath:application_bean_rmdbaccess.xml",
"classpath:application_pm_service_poolmanage.xml"}) // 加载需要的配置配置
@Transactional //事务回滚,便于重复测试
public class TestPoolService{

@Autowired
private IPoolService poolService;

@Mock
private IPoolDao poolDao;

@Before
public void setUp() {
         //可以在每一次单元测试前准备执行一些东西
         MockitoAnnotations.initMocks(this);

//把Spring上下文注入的对象给替换掉
         ReflectionTestUtils.setField(AopTargetUtils.getTarget(poolService), ”poolDao“,poolDao);

/*对void的方法设置模拟*/
Mockito.doAnswer(invocationOnMock -> {
System.out.println("进入了Mock");
return null;
}).when(poolDao).insert(Mockito.any());
}

@Test
public void testAddPool() {

Pool pool = new Pool();
pool.setPoolName("test_pool_001");
pool.setDescription("test add pool");
pool = poolService.addPool(pool);

//进行结果的验证
assertNotNull(pool.getId());
assertNotNull(pool.getPoolCode());
}
}

  ```
与单独使用Mockito相比，最大的不同其实就是在setUp()方法中调用的ReflectionTestUtils.setField(AopTargetUtils.getTarget(poolService), ”poolDao“,poolDao);这个方法。ReflectionTestUtils是Spring-test提供的一个用于反射处理测试类的工具，通过这个，我们可以替换某一个被spring所管理的bean的成员变量。把他换成我们Mock出来的模拟对象。
当然这又引出了一个问题，就是如果依赖的对象的依赖对象需要被Mock，那么手动的不断重复的找需要被Mock的成员变量非常的麻烦。因此，我们可以写一个AbstractTestExecutionListener监听器，当注入依赖的时候，找到被Mock的变量，以及需要被注入的变量，然后做关系的依赖。这样就能自动的对成员变量做替换了。
  ```java
  public class MockitoDependencyInjectionTestExecutionListener extends DependencyInjectionTestExecutionListener {

private static final Map<Class,Object> mockObject = new HashMap<>();

@Override
protected void injectDependencies(final TestContext testContext) throws Exception {
super.injectDependencies(testContext);
init(testContext);
}

private void injectMock(Object bean) throws Exception {

Field[] fields;
/*找到所有的测试用例的字段*/
if (AopUtils.isAopProxy(bean)){
// 如果是代理的话，找到真正的对象

if(AopUtils.isJdkDynamicProxy(bean)) {
Class targetClass = AopTargetUtils.getTarget(bean).getClass();
if (targetClass == null) {
// 可能是远程实现
return;
}
fields = targetClass.getDeclaredFields();
} else { //cglib
/*CGLIB的代理 不支持*/
return;
}

}else {
fields = bean.getClass().getDeclaredFields();
}

List<Field> injectFields = new ArrayList<>();

/*判断字段上的注解*/
for (Field field : fields) {
Annotation[] annotations = field.getAnnotations();
for (Annotation antt : annotations) {
/*如果是Mock字段的,就直接注入Mock的对象*/
if (antt instanceof org.mockito.Mock) {
// 注入mock实例
Object mockObj = mock(field.getType());
mockObject.put(field.getType(),mockObj);
field.setAccessible(true);
field.set(bean, mockObj);
} else if (antt instanceof Autowired) {
/*需要把所有标注为autowired的找到*/
injectFields.add(field);
}
}
}

/*访问每一个被注入的实例*/
for (Field field : injectFields) {
field.setAccessible(true);

/*找到每一个字段的值*/
Object object = field.get(bean);

Class targetClass = field.getType();

if (!replaceInstance(targetClass,bean,field.getName())){
//如果没有被mock过.那么这个字段需要再一次的做递归
injectMock(object);
}
}
}

private boolean replaceInstance(Class targetClass, Object bean, String fieldName) throws Exception {

boolean beMocked = false;
for (Map.Entry<Class, Object> classObjectEntry : mockObject.entrySet()) {
Class type = classObjectEntry.getKey();
if (type.isAssignableFrom(targetClass)){
//如果这个字段是被mock了的对象.那么就使用这个mock的对象来替换
ReflectionTestUtils.setField(AopTargetUtils.getTarget(bean), fieldName, classObjectEntry.getValue());
beMocked = true;
break;
}
}
return beMocked;
}

private void init(final TestContext testContext) throws Exception {
Object bean = testContext.getTestInstance();

injectMock(bean);
}

}

  ```
  以上代码即为Mock初始化的监听器。它会查询这个测试用例的所有的成员变量。找到被标记为@Mock的变量，然后模拟出来。而后，又找到所有被标注为@Autowired的成员变量，判断变量类型是否是需要被Mock的。
当需要使用这个监听器的时候，只需要增加一个注解@TestExecutionListeners即可：
  ```java
  @RunWith(SpringJUnit4ClassRunner.class) // 整合Spring-Test与JUnit
/*如果要加事物,那么当手动设置TestExecutionListeners的时候,需要把TransactionalTestExecutionListener这个也加上*/
@TestExecutionListeners({ MockitoDependencyInjectionTestExecutionListener.class, TransactionalTestExecutionListener.class })
@ContextConfiguration(locations = {"classpath:application_bean_commons.xml",
"classpath:application_bean_rmdbaccess.xml",
"classpath:application_pm_service_poolmanage.xml"}) // 加载需要的配置配置
@Transactional //事务回滚,便于重复测试
public class TestPoolService{

@Autowired
private IPoolService poolService;

@Mock
private IPoolDao poolDao;

@Before
public void setUp() {        
         /*对void的方法设置模拟*/
Mockito.doAnswer(invocationOnMock -> {
System.out.println("进入了Mock");
return null;
}).when(poolDao).insert(Mockito.any());
}

@Test
public void testAddPool() {

Pool pool = new Pool();
pool.setPoolName("test_pool_001");
pool.setDescription("test add pool");
pool = poolService.addPool(pool);

//进行结果的验证
assertNotNull(pool.getId());
assertNotNull(pool.getPoolCode());
}
}

  ```
  
  [1]: http://junit.org/junit4/
  [2]: http://docs.spring.io/spring/docs/4.3.4.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#overview-testing
  [3]: http://ojb6w1er1.bkt.clouddn.com/Mockito1.jpg
  [4]: http://ojb6w1er1.bkt.clouddn.com/Mockito2.jpg