# 设计模式

* idea中查看类的继承关系图

## 代理模式

* 静态代理-切换数据源

  ```java
  @Data
  public class Order {
      private Object orderInfo;
      //订单创建时间进行按年分库
      private Long createTime;
      private String id;
  }
  
  public interface IOrderService {
      int createOrder(Order order);
  }
  
  public class OrderService implements IOrderService {
      private OrderDao orderDao;
      public OrderService(){
          //如果使用Spring应该是自动注入的
          //我们为了使用方便，在构造方法中将orderDao直接初始化了
          orderDao = new OrderDao();
      }
      public int createOrder(Order order) {
          System.out.println("OrderService调用orderDao创建订单");
          return orderDao.insert(order);
      }
  }
  
  public class OrderDao {
      public int insert(Order order){
          System.out.println("OrderDao创建Order成功!");
          return 1;
      }
  }
  
  public class DbRouteProxyTest {
      public static void main(String[] args) {
          try {
              Order order = new Order();
  //            order.setCreateTime(new Date().getTime());
              SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
              Date date = sdf.parse("2017/02/01");
              order.setCreateTime(date.getTime());
              IOrderService orderService = (IOrderService)new OrderServiceDynamicProxy().getInstance(new OrderService());
              orderService.createOrder(order);
          }catch (Exception e){
              e.printStackTrace();
          }
      }
  }
  
  public class DynamicDataSourceEntity {
  
      public final static String DEFAULE_SOURCE = null;
      private final static ThreadLocal<String> local = new ThreadLocal<String>();
      private DynamicDataSourceEntity(){}
  
      public static String get(){return  local.get();}
      public static void restore(){
           local.set(DEFAULE_SOURCE);
      }
      //DB_2018
      //DB_2019
      public static void set(String source){local.set(source);}
      public static void set(int year){local.set("DB_" + year);}
  }
  ```

  ```java
  public class OrderServiceStaticProxy implements IOrderService {
      private SimpleDateFormat yearFormat = new SimpleDateFormat("yyyy");
      private IOrderService orderService;
      public OrderServiceStaticProxy(IOrderService orderService) {
          this.orderService = orderService;
      }
      public int createOrder(Order order) {
          Long time = order.getCreateTime();
          Integer dbRouter = Integer.valueOf(yearFormat.format(new Date(time)));
          System.out.println("静态代理类自动分配到【DB_" +  dbRouter + "】数据源处理数据" );
          DynamicDataSourceEntity.set(dbRouter);
          this.orderService.createOrder(order);
          DynamicDataSourceEntity.restore();
          return 0;
      }
  }
  ```

  ```java
  public class OrderServiceDynamicProxy implements GPInvocationHandler {
      private SimpleDateFormat yearFormat = new SimpleDateFormat("yyyy");
  
      Object proxyObj;
      public Object getInstance(Object proxyObj) {
          this.proxyObj = proxyObj;
          Class<?> clazz = proxyObj.getClass();
          return GPProxy.newProxyInstance(new GPClassLoader(),clazz.getInterfaces(),this);
      }
  
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          before(args[0]);
          Object object = method.invoke(proxyObj,args);
          after();
          return object;
      }
  
      private void after() {
          System.out.println("Proxy after method");
          //还原成默认的数据源
          DynamicDataSourceEntity.restore();
      }
  
      //target 应该是订单对象Order
      private void before(Object target) {
          try {
              //进行数据源的切换
              System.out.println("Proxy before method");
  
              //约定优于配置
              Long time = (Long) target.getClass().getMethod("getCreateTime").invoke(target);
              Integer dbRouter = Integer.valueOf(yearFormat.format(new Date(time)));
              System.out.println("静态代理类自动分配到【DB_" + dbRouter + "】数据源处理数据");
              DynamicDataSourceEntity.set(dbRouter);
          }catch (Exception e){
              e.printStackTrace();
          }
      }
  }
  ```
  
* 动态代理

  * jdk动态代理

    ```java
    public class Girl implements Person {
        public void findLove() {
            System.out.println("高富帅");
            System.out.println("身高180cm");
            System.out.println("有6块腹肌");
        }
    }
    ```

    ```java
    public class JDKMeipo implements InvocationHandler {
    
        private Object target;
        public Object getInstance(Object target) throws Exception{
            this.target = target;
            Class<?> clazz = target.getClass();
            return Proxy.newProxyInstance(clazz.getClassLoader(),clazz.getInterfaces(),this);
        }
    
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            before();
            Object obj = method.invoke(this.target,args);
            after();
           return obj;
        }
    
        private void before(){
            System.out.println("我是媒婆，我要给你找对象，现在已经确认你的需求");
            System.out.println("开始物色");
        }
    
        private void after(){
            System.out.println("OK的话，准备办事");
        }
    }
    
    ```

    ```java
    public class JDKProxyTest {
    
        public static void main(String[] args) {
            try {
                Object obj = new JDKMeipo().getInstance(new Girl());
                Method method = obj.getClass().getMethod("findLove",null);
                method.invoke(obj);
                //obj.findLove();
                //$Proxy0
    //            byte [] bytes = ProxyGenerator.generateProxyClass("$Proxy0",new Class[]{Person.class});
    //            FileOutputStream os = new FileOutputStream("E://$Proxy0.class");
    //            os.write(bytes);
    //            os.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
    ```

  * cglib动态代理

    ```java
    public class Customer {
        public void findLove(){
            System.out.println("儿子要求：肤白貌美大长腿");
        }
    }
    ```

    ```java
    public class CGlibMeipo implements MethodInterceptor {
        public Object getInstance(Class<?> clazz) throws Exception{
            //相当于Proxy，代理的工具类
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(clazz);
            enhancer.setCallback(this);
            return enhancer.create();
        }
    
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            before();
            Object obj = methodProxy.invokeSuper(o,objects);
            after();
            return obj;
        }
    
        private void before(){
            System.out.println("我是媒婆，我要给你找对象，现在已经确认你的需求");
            System.out.println("开始物色");
        }
    
        private void after(){
            System.out.println("OK的话，准备办事");
        }
    }
    ```

    ```java
    public class CglibTest {
        public static void main(String[] args) {
            try {
                //JDK是采用读取接口的信息
                //CGLib覆盖父类方法
                //目的：都是生成一个新的类，去实现增强代码逻辑的功能
                //JDK Proxy 对于用户而言，必须要有一个接口实现，目标类相对来说复杂
                //CGLib 可以代理任意一个普通的类，没有任何要求
                //CGLib 生成代理逻辑更复杂，效率,调用效率更高，生成一个包含了所有的逻辑的FastClass，不再需要反射调用
                //JDK Proxy生成代理的逻辑简单，执行效率相对要低，每次都要反射动态调用
                //CGLib 有个坑，CGLib不能代理final的方法
               System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY,"E://cglib_proxy_classes");
                Customer obj = (Customer) new CGlibMeipo().getInstance(Customer.class);
                System.out.println(obj);
                obj.findLove();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```

  * 自定义动态代理

    ```java
    public class GPMeipo implements GPInvocationHandler {
        private Object target;
        public Object getInstance(Object target) throws Exception{
            this.target = target;
            Class<?> clazz = target.getClass();
            return GPProxy.newProxyInstance(new GPClassLoader(),clazz.getInterfaces(),this);
        }
    
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            before();
            Object obj = method.invoke(this.target,args);
            after();
            return obj;
        }
    
        private void before(){
            System.out.println("我是媒婆，我要给你找对象，现在已经确认你的需求");
            System.out.println("开始物色");
        }
    
        private void after(){
            System.out.println("OK的话，准备办事");
        }
    }
    ```

    ```java
    **
     * 用来生成源代码的工具类
     * Created by Tom.
     */
    public class GPProxy {
    
        public static final String ln = "\r\n";
    
        public static Object newProxyInstance(GPClassLoader classLoader, Class<?> [] interfaces, GPInvocationHandler h){
           try {
               //1、动态生成源代码.java文件
               String src = generateSrc(interfaces);
    
    //           System.out.println(src);
               //2、Java文件输出磁盘
               String filePath = GPProxy.class.getResource("").getPath();
    //           System.out.println(filePath);
               File f = new File(filePath + "$Proxy0.java");
               FileWriter fw = new FileWriter(f);
               fw.write(src);
               fw.flush();
               fw.close();
    
               //3、把生成的.java文件编译成.class文件
               JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
               StandardJavaFileManager manage = compiler.getStandardFileManager(null,null,null);
               Iterable iterable = manage.getJavaFileObjects(f);
    
              JavaCompiler.CompilationTask task = compiler.getTask(null,manage,null,null,null,iterable);
              task.call();
              manage.close();
    
               //4、编译生成的.class文件加载到JVM中来
              Class proxyClass =  classLoader.findClass("$Proxy0");
              Constructor c = proxyClass.getConstructor(GPInvocationHandler.class);
              f.delete();
    
               //5、返回字节码重组以后的新的代理对象
               return c.newInstance(h);
           }catch (Exception e){
               e.printStackTrace();
           }
            return null;
        }
    
        private static String generateSrc(Class<?>[] interfaces){
                StringBuffer sb = new StringBuffer();
                sb.append("package com.gupaoedu.vip.pattern.proxy.dynamicproxy.gpproxy;" + ln);
                sb.append("import com.gupaoedu.vip.pattern.proxy.Person;" + ln);
                sb.append("import java.lang.reflect.*;" + ln);
                sb.append("public class $Proxy0 implements " + interfaces[0].getName() + "{" + ln);
                    sb.append("GPInvocationHandler h;" + ln);
                    sb.append("public $Proxy0(GPInvocationHandler h) { " + ln);
                        sb.append("this.h = h;");
                    sb.append("}" + ln);
                    for (Method m : interfaces[0].getMethods()){
                        Class<?>[] params = m.getParameterTypes();
    
                        StringBuffer paramNames = new StringBuffer();
                        StringBuffer paramValues = new StringBuffer();
                        StringBuffer paramClasses = new StringBuffer();
    
                        for (int i = 0; i < params.length; i++) {
                            Class clazz = params[i];
                            String type = clazz.getName();
                            String paramName = toLowerFirstCase(clazz.getSimpleName());
                            paramNames.append(type + " " +  paramName);
                            paramValues.append(paramName);
                            paramClasses.append(clazz.getName() + ".class");
                            if(i > 0 && i < params.length-1){
                                paramNames.append(",");
                                paramClasses.append(",");
                                paramValues.append(",");
                            }
                        }
    
                        sb.append("public " + m.getReturnType().getName() + " " + m.getName() + "(" + paramNames.toString() + ") {" + ln);
                            sb.append("try{" + ln);
                                sb.append("Method m = " + interfaces[0].getName() + ".class.getMethod(\"" + m.getName() + "\",new Class[]{" + paramClasses.toString() + "});" + ln);
                                sb.append((hasReturnValue(m.getReturnType()) ? "return " : "") + getCaseCode("this.h.invoke(this,m,new Object[]{" + paramValues + "})",m.getReturnType()) + ";" + ln);
                            sb.append("}catch(Error _ex) { }");
                            sb.append("catch(Throwable e){" + ln);
                            sb.append("throw new UndeclaredThrowableException(e);" + ln);
                            sb.append("}");
                            sb.append(getReturnEmptyCode(m.getReturnType()));
                        sb.append("}");
                    }
                sb.append("}" + ln);
                return sb.toString();
        }
    
    
        private static Map<Class,Class> mappings = new HashMap<Class, Class>();
        static {
            mappings.put(int.class,Integer.class);
        }
    
        private static String getReturnEmptyCode(Class<?> returnClass){
            if(mappings.containsKey(returnClass)){
                return "return 0;";
            }else if(returnClass == void.class){
                return "";
            }else {
                return "return null;";
            }
        }
    
        private static String getCaseCode(String code,Class<?> returnClass){
            if(mappings.containsKey(returnClass)){
                return "((" + mappings.get(returnClass).getName() +  ")" + code + ")." + returnClass.getSimpleName() + "Value()";
            }
            return code;
        }
    
        private static boolean hasReturnValue(Class<?> clazz){
            return clazz != void.class;
        }
    
        private static String toLowerFirstCase(String src){
            char [] chars = src.toCharArray();
            chars[0] += 32;
            return String.valueOf(chars);
        }
    
    }
    ```

    ```java
    public class GPClassLoader extends ClassLoader {
    
        private File classPathFile;
        public GPClassLoader(){
            String classPath = GPClassLoader.class.getResource("").getPath();
            this.classPathFile = new File(classPath);
        }
    
        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
    
            String className = GPClassLoader.class.getPackage().getName() + "." + name;
            if(classPathFile  != null){
                File classFile = new File(classPathFile,name.replaceAll("\\.","/") + ".class");
                if(classFile.exists()){
                    FileInputStream in = null;
                    ByteArrayOutputStream out = null;
                    try{
                        in = new FileInputStream(classFile);
                        out = new ByteArrayOutputStream();
                        byte [] buff = new byte[1024];
                        int len;
                        while ((len = in.read(buff)) != -1){
                            out.write(buff,0,len);
                        }
                        return defineClass(className,out.toByteArray(),0,out.size());
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
            return null;
        }
    }
    ```

    ```java
    public interface GPInvocationHandler {
        public Object invoke(Object proxy, Method method, Object[] args)
                throws Throwable;
    }
    ```

    ```java
    public class GPProxyTest {
        public static void main(String[] args) {
            try {
                //JDK动态代理的实现原理
                Person obj = (Person) new GPMeipo().getInstance(new Girl());
                System.out.println(obj.getClass());
                obj.findLove();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
    ```

## 模版方法模式

```java
@Data
public class Member {
    private String username;
    private String password;
    private String nickname;
    private int age;
    private String addr;
}
```

```java
public class MemberDaoTest {
    public static void main(String[] args) {
        MemberDao memberDao = new MemberDao(null);
        List<?> result = memberDao.selectAll();
        System.out.println(result);
    }
}
```

```java
/**
 * ORM映射定制化的接口
 */
public interface RowMapper<T> {
    T mapRow(ResultSet rs,int rowNum) throws Exception;
}
```

```java
public abstract class JdbcTemplate {
    private DataSource dataSource;

    public JdbcTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public List<?> executeQuery(String sql, RowMapper<?> rowMapper, Object[] values){
        try {
            //1、获取连接
            Connection conn = this.getConnection();
            //2、创建语句集
            PreparedStatement pstm = this.createPrepareStatement(conn,sql);
            //3、执行语句集
            ResultSet rs = this.executeQuery(pstm,values);
            //4、处理结果集
            List<?> result = this.paresResultSet(rs,rowMapper);
            //5、关闭结果集
            this.closeResultSet(rs);
            //6、关闭语句集
            this.closeStatement(pstm);
            //7、关闭连接
            this.closeConnection(conn);
            return result;
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }

    protected void closeConnection(Connection conn) throws Exception {
        //数据库连接池，我们不是关闭
        conn.close();
    }

    protected void closeStatement(PreparedStatement pstm) throws Exception {
        pstm.close();
    }

    protected void closeResultSet(ResultSet rs) throws Exception {
        rs.close();
    }

    protected List<?> paresResultSet(ResultSet rs, RowMapper<?> rowMapper) throws Exception {
        List<Object> result = new ArrayList<Object>();
        int rowNum = 1;
        while (rs.next()){
            result.add(rowMapper.mapRow(rs,rowNum ++));
        }
        return result;
    }

    protected ResultSet executeQuery(PreparedStatement pstm, Object[] values) throws Exception {
        for (int i = 0; i < values.length; i++) {
            pstm.setObject(i,values[i]);
        }
        return pstm.executeQuery();
    }

    protected PreparedStatement createPrepareStatement(Connection conn, String sql) throws Exception {
        return conn.prepareStatement(sql);
    }

    public Connection getConnection() throws Exception {
        return this.dataSource.getConnection();
    }
}
```

```java
public class MemberDao extends JdbcTemplate {
    public MemberDao(DataSource dataSource) {
        super(dataSource);
    }

    public List<?> selectAll(){
        String sql = "select * from t_member";
        return super.executeQuery(sql, new RowMapper<Member>() {
            public Member mapRow(ResultSet rs, int rowNum) throws Exception {
                Member member = new Member();
                //字段过多，原型模式
                member.setUsername(rs.getString("username"));
                member.setPassword(rs.getString("password"));
                member.setAge(rs.getInt("age"));
                member.setAddr(rs.getString("addr"));
                return member;
            }
        },null);
    }
}
```

## 策略+适配器+简单工厂

```java
/**
 * 在适配器里面，这个接口是可有可无，不要跟模板模式混淆
 * 模板模式一定是抽象类，而这里仅仅只是一个接口
 */
public interface LoginAdapter {
    boolean support(Object adapter);
    ResultMsg login(String id,Object adapter);
}
```

```java
public class LoginForQQAdapter implements LoginAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForQQAdapter;
    }

    public ResultMsg login(String id, Object adapter) {
        return null;
    }
}
```

```java
public class LoginForSinaAdapter implements LoginAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForSinaAdapter;
    }
    public ResultMsg login(String id, Object adapter) {
        return null;
    }
}
```

```java
public class LoginForTelAdapter implements LoginAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForTelAdapter;
    }
    public ResultMsg login(String id, Object adapter) {
        return null;
    }
}
```

```java
public class LoginForTokenAdapter implements LoginAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForTokenAdapter;
    }
    public ResultMsg login(String id, Object adapter) {
        return null;
    }
}
```

```java
public class LoginForWechatAdapter implements LoginAdapter {
    public boolean support(Object adapter) {
        return adapter instanceof LoginForWechatAdapter;
    }
    public ResultMsg login(String id, Object adapter) {
        return null;
    }
}
```

```java
public interface RegistAdapter {
    boolean support(Object adapter);
    ResultMsg login(String id, Object adapter);
}
```

```java
public class RegistForQQAdapter implements RegistAdapter,LoginAdapter {
    public boolean support(Object adapter) {
        return false;
    }

    public ResultMsg login(String id, Object adapter) {
        return null;
    }
}
```

```java
// 只扩展
public interface IPassportForThird {
    // QQ登录
    ResultMsg loginForQQ(String id);
    // 微信登录
    ResultMsg loginForWechat(String id);
    // 记住登录状态后自动登录
    ResultMsg loginForToken(String token);
    // 手机号登录
    ResultMsg loginForTelphone(String telphone, String code);
    // 注册后自动登录
    ResultMsg loginForRegist(String username, String passport);
}
```

```java
// 结合策略模式、工厂模式、适配器模式
public class PassportForThirdAdapter extends SiginService implements IPassportForThird {

    public ResultMsg loginForQQ(String id) {
//        return processLogin(id,RegistForQQAdapter.class);
        return processLogin(id,LoginForQQAdapter.class);
    }

    public ResultMsg loginForWechat(String id) {
        return processLogin(id,LoginForWechatAdapter.class);
    }

    public ResultMsg loginForToken(String token) {
        return processLogin(token,LoginForTokenAdapter.class);
    }

    public ResultMsg loginForTelphone(String telphone, String code) {
        return processLogin(telphone,LoginForTelAdapter.class);
    }

    public ResultMsg loginForRegist(String username, String passport) {
        super.regist(username,passport);
        return super.login(username,passport);
    }

    private ResultMsg processLogin(String key,Class<? extends LoginAdapter> clazz){
        try{
            //适配器不一定要实现接口
            LoginAdapter adapter = clazz.newInstance();

            //判断传过来的适配器是否能处理指定的逻辑
            if(adapter.support(adapter)){
                return adapter.login(key,adapter);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
    //类图的快捷键  Ctrl + Alt + Shift + U
}
```

```java
public class PassportTest {
    public static void main(String[] args) {
        IPassportForThird passportForThird = new PassportForThirdAdapter();
        passportForThird.loginForQQ("");
    }
}
```

## 装饰器模式

```java
@Data
public class Member {
    private String username;
    private String password;
    private String mid;
    private String info;
}

@Data
public class ResultMsg {

    private int code;
    private String msg;
    private Object data;

    public ResultMsg(int code, String msg, Object data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```

```java
public interface ISigninService {
    ResultMsg regist(String username, String password);
    /**
     * 登录的方法
     */
    ResultMsg login(String username, String password);
}

public class SigninService implements ISigninService {
    public ResultMsg regist(String username,String password){
        return  new ResultMsg(200,"注册成功",new Member());
    }

    /**
     * 登录的方法
     */
    public ResultMsg login(String username,String password){
        return null;
    }
}
```

```java
public interface ISiginForThirdService extends ISigninService {
    // QQ登录
    ResultMsg loginForQQ(String id);
    // 微信登录
    ResultMsg loginForWechat(String id);
    // 记住登录状态后自动登录
    ResultMsg loginForToken(String token);
    // 手机号登录
    ResultMsg loginForTelphone(String telphone, String code);
    // 注册后自动登录
    ResultMsg loginForRegist(String username, String passport);
}
```

```java
public class SiginForThirdService implements ISiginForThirdService {

    private ISigninService signinService;

    public SiginForThirdService(ISigninService signinService) {
        this.signinService = signinService;
    }

    public ResultMsg regist(String username, String password) {
        return signinService.regist(username,password);
    }

    public ResultMsg login(String username, String password) {
        return signinService.login(username,password);
    }

    public ResultMsg loginForQQ(String id) {
        return null;
    }

    public ResultMsg loginForWechat(String id) {
        return null;
    }

    public ResultMsg loginForToken(String token) {
        return null;
    }

    public ResultMsg loginForTelphone(String telphone, String code) {
        return null;
    }

    public ResultMsg loginForRegist(String username, String passport) {
        return null;
    }
}
```

```java
public class DecoratorTest {
    public static void main(String[] args) {
        //满足一个is-a
        ISiginForThirdService siginForThirdService = new SiginForThirdService(new SigninService());
        siginForThirdService.loginForQQ("sdfasfdasfsf");
    }
}
```

## 观察者模式

```java
/**
 * 监听器的一种包装,标准事件源格式的定义
 */
public class Event {
    //事件源，事件是由谁发起的保存起来
    private Object source;
    //事件触发，要通知谁
    private Object target;
    //事件触发，要做什么动作，回调
    private Method callback;
    //事件的名称，触发的是什么事件
    private String trigger;
    //事件触发的时间
    private long time;

    public Event(Object target, Method callback) {
        this.target = target;
        this.callback = callback;
    }

    public Event setSource(Object source) {
        this.source = source;
        return this;
    }

    public Event setTime(long time) {
        this.time = time;
        return this;
    }

    public Object getSource() {
        return source;
    }

    public Event setTrigger(String trigger) {
        this.trigger = trigger;
        return this;
    }

    public long getTime() {
        return time;
    }

    public Object getTarget() {
        return target;
    }

    public Method getCallback() {
        return callback;
    }

    @Override
    public String toString() {
        return "Event{" + "\n" +
                "\tsource=" + source.getClass() + ",\n" +
                "\ttarget=" + target.getClass() + ",\n" +
                "\tcallback=" + callback + ",\n" +
                "\ttrigger='" + trigger + "',\n" +
                "\ttime=" + time + "'\n" +
                '}';
    }
}
```

```java
/**
 * 监听器，它就是观察者
 */
public class EventLisenter {

    //JDK底层的Lisenter通常也是这样来设计的
    protected Map<String,Event> events = new HashMap<String,Event>();

    //事件名称和一个目标对象来触发事件
    public void addLisenter(String eventType,Object target){
        try {
            this.addLisenter(
                    eventType,
                    target,
                    target.getClass().getMethod("on" + toUpperFirstCase(eventType),Event.class));
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public void addLisenter(String eventType,Object target,Method callback){
        //注册事件
        events.put(eventType, new Event(target, callback));
    }


    //触发，只要有动作就触发
    private void trigger(Event event) {
        event.setSource(this);
        event.setTime(System.currentTimeMillis());

        try {
            //发起回调
            if(event.getCallback() != null){
                //用反射调用它的回调函数
                event.getCallback().invoke(event.getTarget(),event);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //事件名称触发
    protected void trigger(String trigger){
        if(!this.events.containsKey(trigger)){return;}
        trigger(this.events.get(trigger).setTrigger(trigger));
    }

    //逻辑处理的私有方法，首字母大写
    private String toUpperFirstCase(String str){
        char[] chars = str.toCharArray();
        chars[0] -= 32;
        return String.valueOf(chars);
    }
}
```

```java
public class Keybord extends EventLisenter {
    public void down(){
    }
    public void up(){
    }
}

public class Mouse extends EventLisenter {

    public void click(){
        System.out.println("调用单击方法");
        this.trigger(MouseEventType.ON_CLICK);
    }

    public void doubleClick(){
        System.out.println("调用双击方法");
        this.trigger(MouseEventType.ON_DOUBLE_CLICK);
    }

    public void up(){
        System.out.println("调用弹起方法");
        this.trigger(MouseEventType.ON_UP);
    }

    public void down(){
        System.out.println("调用按下方法");
        this.trigger(MouseEventType.ON_DOWN);
    }

    public void move(){
        System.out.println("调用移动方法");
        this.trigger(MouseEventType.ON_MOVE);
    }

    public void wheel(){
        System.out.println("调用滚动方法");
        this.trigger(MouseEventType.ON_WHEEL);
    }

    public void over(){
        System.out.println("调用悬停方法");
        this.trigger(MouseEventType.ON_OVER);
    }

    public void blur(){
        System.out.println("调用获焦方法");
        this.trigger(MouseEventType.ON_BLUR);
    }

    public void focus(){
        System.out.println("调用失焦方法");
        this.trigger(MouseEventType.ON_FOCUS);
    }
}
```

```java
/**
 * 自己写的逻辑，用于回调
 */
public class MouseEventCallback {

    public void onClick(Event e){
        System.out.println("===========触发鼠标单击事件==========" + "\n" + e);
    }

    public void onDoubleClick(Event e){
        System.out.println("===========触发鼠标双击事件==========" + "\n" + e);
    }

    public void onUp(Event e){
        System.out.println("===========触发鼠标弹起事件==========" + "\n" + e);
    }

    public void onDown(Event e){
        System.out.println("===========触发鼠标按下事件==========" + "\n" + e);
    }

    public void onMove(Event e){
        System.out.println("===========触发鼠标移动事件==========" + "\n" + e);
    }

    public void onWheel(Event e){
        System.out.println("===========触发鼠标滚动事件==========" + "\n" + e);
    }

    public void onOver(Event e){
        System.out.println("===========触发鼠标悬停事件==========" + "\n" + e);
    }

    public void onBlur(Event e){
        System.out.println("===========触发鼠标失焦事件==========" + "\n" + e);
    }

    public void onFocus(Event e){
        System.out.println("===========触发鼠标获焦事件==========" + "\n" + e);
    }
}
```

```java
public interface MouseEventType {
    //单击
    String ON_CLICK = "click";
    //双击
    String ON_DOUBLE_CLICK = "doubleClick";
    //弹起
    String ON_UP = "up";
    //按下
    String ON_DOWN = "down";
    //移动
    String ON_MOVE = "move";
    //滚动
    String ON_WHEEL = "wheel";
    //悬停
    String ON_OVER = "over";
    //失焦
    String ON_BLUR = "blur";
    //获焦
    String ON_FOCUS = "focus";
}
```

```java
public class MouseEventTest {
    public static void main(String[] args) {
        MouseEventCallback callback = new MouseEventCallback();
        Mouse mouse = new Mouse();
        //@谁？  @回调方法
        mouse.addLisenter(MouseEventType.ON_CLICK,callback);
        mouse.addLisenter(MouseEventType.ON_FOCUS,callback);
        mouse.click();
        mouse.focus();
    }
}
```

## 观察者模式-guava

```java
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>20.0</version>
</dependency>
  
public class GuavaEvent {
    @Subscribe
    public void subscribe(String str){
        System.out.println("执行subscribe方法，传入的参数是：" + str);
    }
}

public class GuavaEventTest {
    public static void main(String[] args) {
        //消息总线
        EventBus eventBus = new EventBus();
        GuavaEvent guavaEvent = new GuavaEvent();
        eventBus.register(guavaEvent);
        eventBus.post("Tom");

        //从Struts到SpringMVC的升级
        //因为Struts面向的类，而SpringMVC面向的是方法

        //前面两者面向的是类，Guava面向是方法

        //能够轻松落地观察模式的一种解决方案
        //MQ
    }
}
```

# 多线程

## 模拟高并发

```java
public class ConcurrentExecutor {
    /**
     * @param runHandler
     * @param executeCount 发起请求总数
     * @param concurrentCount 同时并发执行的线程数
     * @throws Exception
     */
    public static void execute(final RunHandler runHandler,int executeCount,int concurrentCount) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        //控制信号量，此处用于控制并发的线程数
        final Semaphore semaphore = new Semaphore(concurrentCount);
        //闭锁，可实现计数量递减
        final CountDownLatch countDownLatch = new CountDownLatch(executeCount);
        for (int i = 0; i < executeCount; i ++){
            executorService.execute(new Runnable() {
                public void run() {
                    try{
                        //执行此方法用于获取执行许可，当总计未释放的许可数不超过executeCount时,
                        //则允许同性，否则线程阻塞等待，知道获取到许可
                        semaphore.acquire();
                        runHandler.handler();
                        //释放许可
                        semaphore.release();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                    countDownLatch.countDown();
                }
            });
        }
        countDownLatch.await();//线程阻塞，知道闭锁值为0时，阻塞才释放，继续往下执行
        executorService.shutdown();
    }
    public interface RunHandler{
        void handler();
    }
}
```

## 线程池并发同步

* 组装一个对象，这个对象由大量小的内容组成，这些内容是无关联无依赖关系的，如果我们串行的去执行，如果每个任务耗时10秒钟，一共有10个任务，那我们就需要100秒才能获取到结果。显然我们可以采用线程池，每个任务起一个线程，忽略线程启动时间，我们只需要10秒钟就能获取到结果

  ```java
  // 这是任务的抽象
  class GetContentTask implements Callable<String> {
  		
  		private String name;
  		private Integer sleepTimes;
  		
  		public GetContentTask(String name, Integer sleepTimes) {
  			this.name = name;
  			this.sleepTimes = sleepTimes;
  		}
  		public String call() throws Exception {
  			// 假设这是一个比较耗时的操作
  			Thread.sleep(sleepTimes * 1000);
  			return "this is content : hello " + this.name;
  		}	
  }
  ```

  ```java
  // 方法一
  ExecutorService executorService = Executors.newCachedThreadPool();
  CompletionService<String> completionService = new ExecutorCompletionService(executorService);
  ExecuteServiceDemo executeServiceDemo = new ExecuteServiceDemo();
  // 十个
  long startTime = System.currentTimeMillis();
  int count = 0;
  for (int i = 0;i < 10;i ++) {
    count ++;
    GetContentTask getContentTask = new ExecuteServiceDemo.GetContentTask("micro" + i, 10);
    completionService.submit(getContentTask);
  }
  System.out.println("提交完任务，主线程空闲了, 可以去做一些事情。");
  // 假装做了8秒种其他事情
  try {
    Thread.sleep(8000);
    System.out.println("主线程做完了，等待结果");
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
  try {
    // 做完事情要结果
    for (int i = 0;i < count;i ++) {
      Future<String> result = completionService.take();
      System.out.println(result.get());
    }
    long endTime = System.currentTimeMillis();
    System.out.println("耗时 : " + (endTime - startTime) / 1000);
  }  catch (Exception ex) {
    System.out.println(ex.getMessage());
  }
  ```

  ```shell
  提交完任务，主线程空闲了, 可以去做一些事情。
  主线程做完了，等待结果
  this is content : hello micro9
  this is content : hello micro7
  this is content : hello micro2
  this is content : hello micro5
  this is content : hello micro4
  this is content : hello micro8
  this is content : hello micro1
  this is content : hello micro3
  this is content : hello micro0
  this is content : hello micro6
  耗时 : 10
  ```

  ```java
  // 如果多个不想一个一个提交，可以采用 invokeAll一并提交，但是会同步等待这些任务
  // 方法二
  ExecutorService executeService = Executors.newCachedThreadPool();
  List<GetContentTask> taskList = new ArrayList<GetContentTask>();
  long startTime = System.currentTimeMillis();
  for (int i = 0;i < 10;i ++) {
    taskList.add(new GetContentTask("micro" + i, 10));
  }
  try {
    System.out.println("主线程发起异步任务请求");
    List<Future<String>> resultList = executeService.invokeAll(taskList);
    // 这里会阻塞等待resultList获取到所有异步执行的结果才会执行 
    for (Future<String> future : resultList) {
      System.out.println(future.get());
    }
    // 主线程假装很忙执行8秒钟
    Thread.sleep(8);
    long endTime = System.currentTimeMillis();
    System.out.println("耗时 : " + (endTime - startTime) / 1000);
  } catch (Exception e) {
    e.printStackTrace();
  }
  ```

  ```shell
  主线程发起异步任务请求
  this is content : hello micro0
  this is content : hello micro1
  this is content : hello micro2
  this is content : hello micro3
  this is content : hello micro4
  this is content : hello micro5
  this is content : hello micro6
  this is content : hello micro7
  this is content : hello micro8
  this is content : hello micro9
  耗时 : 10
  ```

  ```java
  // 方法三
  // 如果一系列请求，我们并不需要等待每个请求，我们可以invokeAny,只要某一个请求返回即可
  ExecutorService executorService = Executors.newCachedThreadPool();
  ArrayList<GetContentTask> taskList = new ArrayList<GetContentTask>();
  taskList.add(new GetContentTask("micro1",3));
  taskList.add(new GetContentTask("micro2", 6));
  try {
    List<Future<String>> resultList = executorService.invokeAll(taskList);// 等待6秒 
    //			String result2 = executorService.invokeAny(taskList); // 等待3秒
    // invokeAll 提交一堆任务并行处理并拿到结果
    // invokeAny就是提交一堆并行任务拿到一个结果即可
    for (Future<String> result : resultList) {
      System.out.println(result.get());
    }
    //			System.out.println(result2);
  } catch (Exception e) {
    e.printStackTrace();
  }
  System.out.println("主线程等待");
  
  ```

  ```java
  // 如果我虽然发送了一堆异步的任务，但是我只等待一定的时间，在规定的时间没有返回我就不要了，例如很多时候的网络请求其他服务器如果要数据，由于网络原因不能一直等待，在规定时间内去拿，拿不到就我使用一个默认值。这样的场景，我们可以使用下面的写法：
  try {
    ExecutorService executorService = Executors.newCachedThreadPool();
    List<Callable<String>> taskList = new ArrayList<Callable<String>>();
    taskList.add(new GetContentTask("micro1", 4));
    taskList.add(new GetContentTask("micro2", 6));
    // 等待五秒
    List<Future<String>> resultList = executorService.invokeAll(taskList, 5, TimeUnit.SECONDS);
    for (Future<String> future : resultList) {
      System.out.println(future.get());
    }
  } catch (Exception e) {
    e.printStackTrace();
  } 
  // 结果
  this is content : hello micro1
  java.util.concurrent.CancellationException
  	at java.util.concurrent.FutureTask.report(FutureTask.java:121)
  	at java.util.concurrent.FutureTask.get(FutureTask.java:192)
  	at com.micro.demo.spring.ExecuteServiceDemo.main(ExecuteServiceDemo.java:105)
  // 因为只等待5秒，6秒的那个任务自然获取不到，抛出异常，如果将等待时间设置成8秒，就都能获取到  
  ```

# 反射

```java
// 简单工厂模式
public ICourse create(Class<? extends ICourse> clazz){ 
  try {
		if (null != clazz) {
			return clazz.newInstance();
		}
    }catch (Exception e){
       e.printStackTrace();
    }
			return null;
}

public static void main(String[] args) { 
  CourseFactory factory = new CourseFactory(); 
  ICourse course = factory.create(JavaCourse.class); 
  course.record();
}
```

# JSON

```java
//创建json对象
1. JSONObject jo = new JSONObject();
   jo.put("name", "jon doe");
2. new JSONObject(Map);
3. new JSONObject(Object);
4. new JSONObject("{\"city\":\"chicago\",\"name\":\"jon doe\",\"age\":\"22\"}");

//创建json数组
1. JSONArray ja = new JSONArray();
   ja.put(Boolean.TRUE);
   ja.put(Object)
2. new JSONArray(List);
2. JSONArray ja = new JSONArray("[true, \"lorem ipsum\", 215]");

//json字符串转json对象
1. String json = "{\"2\":\"efg\",\"1\":\"abc\"}";   
   JSONObject json_test = JSONObject.fromObject(json); 
JSONObject jsonObject = JSON.parseObject(str);

//@JSONField
//import com.alibaba.fastjson.annotation.JSONField;
1. 指定字段的名称
	@JSONField(name="role_name")  
	private String roleName;
2. 使用format制定日期格式
	// 配置date序列化和反序列使用yyyyMMdd日期格式  
	@JSONField(format="yyyyMMdd")  
	public Date date; 
3. 指定字段的顺序
	@JSONField(ordinal = 3)  
   private int f0;  
4. 使用serialize/deserialize指定字段不序列化
	@JSONField(serialize=false)
	public Date date;
```

```java
// json文件 -> json对象 -> 字符串 -> list
String jsonStr = JSON.toJSONString(JSON.parse(file.getBytes()));
List<ResDataVO> resData = JSON.parseObject(jsonStr, new TypeReference<List<ResDataVO>>(){});
// json字符串 转 json对象
// json字符串 转 java对象
ResDataVO res = JSON.parseObject(String.valueOf(obj), ResDataVO.class);
ResDataVO resDataVO = JSONObject.parseObject(String.valueOf(obj), ResDataVO.class);
// Object 转 JSONObject
(JSONObject) JSONObject.toJSON(obj);
// Java对象 转 json字符串
String jsonStr = JSON.toJSONString(obj);
// null
Object obj = null;
JSONObject object = (JSONObject)obj;  // null
//
JSONObject jsonObj = new JSONObject(jsonStr);
ExamPaper examPaper = JSONObject.parseObject(parameters,ExamPaper.class);
// json文件转为对象
ResExportResultConciseBean resBean = (ResExportResultConciseBean)JSONObject.parseObject(file.getBytes(), ResExportResultConciseBean.class);
```

# Lambda

* list属性值求和

  ```java
  int accessCount = resList.stream().mapToInt(XDIndictorInfoRelease::getAccess).sum();
  ```

* 获取list某个属性

  ```java
  Set<String> dataId = resList.stream().map(XDIndictorInfoRelease::getDataid).collect(Collectors.toSet());
  ```

* 提取某List对象中的几个属性去组成另一个List对象

  ```java
  List<DecreaseStockInput> decreaseStockInputList = orderDTO.getOrderDetailList().stream()
    .map(e -> new DecreaseStockInput(e.getProductId(), e.getProductQuantity()))
    .collect(Collectors.toList());
  ```

* groupingBy

  ```java
  // 按多个属性进行分组
  Map<Pair<String, String>, List<Test>> collect = test.stream().collect(Collectors.groupingBy(t -> Pair.of(t.getNameA(), t.getNameB())));
  ```

* foreach设置属性值

  ```java
  productInfoPage.getContent().stream().forEach(e -> e.addImageHost(upYunConfig.getImageHost()));
  ```

* list转字符串

  ```java
  String collect = list.stream().collect(Collectors.joining(","));
  // 或
  String joinStr = String.join(",", roleNames);
  ```

* filter

  ```java
  // 条件判断并求和
  list.stream().filter(i -> i > 10).mapToInt(i -> i).sum();
// 创建一个字符串列表，每个字符串长度大于2
  List<String> filtered = strList.stream().filter(x -> x.length()> 2).collect(Collectors.toList());
  ```
  
* 最大值、最小值、平均值、总和

  ```java
  //获取数字的个数、最小值、最大值、总和以及平均值
  List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
  IntSummaryStatistics stats = primes.stream().mapToInt((x) -> x).summaryStatistics();
  System.out.println("Highest prime number in List : " + stats.getMax());
  System.out.println("Lowest prime number in List : " + stats.getMin());
  System.out.println("Sum of all prime numbers : " + stats.getSum());
  System.out.println("Average of all prime numbers : " + stats.getAverage());
  ```

* 实现runnable

  ```java
  new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
  ```

* 处理事件

  ```java
  show.addActionListener((e) -> {
      System.out.println("Light, Camera, Action !! Lambda expressions Rocks");
  });
  ```

* 迭代

  ```java
  List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
  features.forEach(n -> System.out.println(n));
  //features.forEach(System.out::println);
  ```

* 与**Predicate**接口配合，**Predicate**接口适合过滤

  ```java
  // 可以用and()、or()逻辑函数来合并Predicate
  // 例如要找到所有以J开始，长度为四个字母的名字，可合并两个Predicate并传入
  Predicate<String> startsWithJ = (n) -> n.startsWith("J");
  Predicate<String> fourLetterLong = (n) -> n.length() == 4;
  names.stream()
      .filter(startsWithJ.and(fourLetterLong))
      .forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n)); 
  ```

* map允许将对象进行转换，**reduce()** 函数可以将所有值合并成一个

  ```java
  List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
  costBeforeTax.stream().map((cost) -> cost + .12*cost).forEach(System.out::println);
  
  // 将字符串换成大写并用逗号链接起来
  List<String> G7 = Arrays.asList("USA", "Japan", "France", "Germany", "Italy", "U.K.","Canada");
  String G7Countries = G7.stream().map(x -> x.toUpperCase()).collect(Collectors.joining(", "));
  
  // 去重
  List<Integer> numbers = Arrays.asList(9, 10, 3, 4, 7, 3, 4);
  List<Integer> distinct = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
  
  List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
  double bill = costBeforeTax.stream().map((cost) -> cost + .12*cost).reduce((sum, cost) -> sum + cost).get();
  System.out.println("Total : " + bill); 
  ```

# 加解密

* [md5](https://blog.csdn.net/junmoxi/article/details/80841555)
* [sm2、3、4](https://github.com/xjfme/SM2_SM3_SM4Encrypt)
* [aes](https://blog.csdn.net/u011781521/article/details/77932321)
* [rsa](https://blog.csdn.net/defonds/article/details/42775183)

# File

```java
File directory = new File("");
sString workspacePath = directory.getCanonicalPath(); //获取工作空间的绝对路径
```

# SpringBoot添加本地依赖

```xml
<!--将依赖放入main目录下的lib文件夹中-->
<dependency>
  <groupId>com.tencent.autocloud.compliance</groupId>
  <artifactId>code-checker</artifactId>
  <version>${version}</version>
  <scope>system</scope>
  <systemPath>${basedir}/src/main/lib/code-checker-0.1.7-SNAPSHOT.jar</systemPath>
  <exclusions>
    <exclusion>
      <artifactId>lombok</artifactId>
      <groupId>org.projectlombok</groupId>
    </exclusion>
    <exclusion>
      <artifactId>junit</artifactId>
      <groupId>junit</groupId>
    </exclusion>
  </exclusions>
</dependency>
```

# 其他

* jdbc

  ```java
  public void save(Student stu){
  	String sql="INSERT INTO t_student(name,age) VALUES(?,?)"; 
    Connection conn=null;
  	Statement st=null;
  	try{
  		// 1. 加载注册驱动
  		Class.forName("com.mysql.jdbc.Driver");
  		// 2. 获取数据库连接 
      conn=DriverManager.getConnection("jdbc:mysql:///jdbcdemo","root","root"); 
      // 3. 创建语句对象
  		PreparedStatement ps=conn.prepareStatement(sql); 
      ps.setObject(1,stu.getName());
  		ps.setObject(2,stu.getAge());
  		// 4. 执行 SQL 语句
  		ps.executeUpdate();
  		// 5. 释放资源
    }catch(Exception e){
         e.printStackTrace();
      }finally{
         try{
             if(st!=null)
                st.close();
  			  }catch(SQLException e){ 
           	e.printStackTrace();
          }finally{
             try{
                 if(conn!=null)
                    conn.close();
  						}catch(SQLException e){ 
               		e.printStackTrace();
  						} 
         }
  		} 
  }
  ```

* JdbcTemplate

  ```java
  // 增加
  String sql = "INSERT INTO t_student(name,age) VALUES(?,?)"; 
  Object[] params=new Object[]{stu.getName(),stu.getAge()}; 
  JdbcTemplate.update(sql, params);
  // 删除
  String sql = "DELETE FROM t_student WHERE id = ?";
  JdbcTemplate.update(sql, id);
  // 修改
  String sql = "UPDATE t_student SET name = ?,age = ? WHERE id = ?"; 
  Object[] params=new Object[]{stu.getName(),stu.getAge(),stu.getId()}; 
  JdbcTemplate.update(sql, params);
  // 查询
  String sql = "SELECT * FROM t_student WHERE id=?"; 
  List<Student> list = JDBCTemplate.query(sql, id); 
  return list.size()>0? list.get(0):null;
  // 查询
  String sql = "SELECT * FROM t_student "; 
  List<Student> list = JDBCTemplate.query(sql);
  ```

* 获取properties中属性值

  ```java
  ClassLoader loader = Thread.currentThread().getContextClassLoader(); 
  InputStream inputStream = loader.getResourceAsStream("db.properties"); 
  p = new Properties();
  p.load(inputStream);
  p.getProperty("driverClassName");
  ```

