# 1 、什么是Mybatis

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516173625701.png" alt="image-20200516173625701" style="zoom:67%;" />

- Mybatis是一款优秀的**持久层框架**
- 它支持定制化SQL,存储过程以及高级映射
- Mybatis避免了几乎所有的JDBC代码和手动设置参数已经获取结果集
- Mybatis可以使用简单的XML或者注解来配置和映射原生类型，接口和Java的POJO为数据库中的对象



# 2、第一个Mybatis程序

1. 导入mybatis和mysql驱动依赖

   ```xml
    <!--导入依赖-->
       <dependencies>
           <!--mysql的驱动-->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.47</version>
           </dependency>
               <!--mybatis的依赖-->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.2</version>
           </dependency>
       </dependencies>
   ```

2. 编写Mybatis的核心配置文件**mybatis-config.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/mydatabase?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8"/>
                   <property name="username" value="root"/>
                   <property name="password" value="686995"/>
               </dataSource>
           </environment>
       </environments>
   </configuration>
   ```

3. 编写Mybatis工具类，方便获取sqlSession

   ```java
   /* Mabatis工具类
    */
   public class MybatisUtils {
   
       private static SqlSessionFactory sqlSessionFactory;
   
       /* 在类被初始化的时候创建sqlSessionFactory
        */
       static {
           try {
               String resource = "org/mybatis/example/mybatis-config.xml";
               InputStream input = Resources.getResourceAsStream("mybatis-config.xml");
               sqlSessionFactory = new SqlSessionFactoryBuilder().build(input);
           } catch (IOException e) {
               System.out.println("未找到Mabtis核心配置文件！");
           }
       }
   
       /* 提供获取sqlSession的方法
        * sqlSessionFactory -> sqlSession
        */
       public static SqlSession getSqlSession() {
           return sqlSessionFactory.openSession();
       }
   }
   ```

   # 2、第一个Mybatis程序
   
   2、第一个Mybatis程序
   
   **步骤：**
   
   1. 导入相关jar包
   
      ```xml
       		<!-- spring的依赖 -->
              <dependency>
                  <groupId>org.springframework</groupId>
                  <artifactId>spring-webmvc</artifactId>
                  <version>5.2.1.RELEASE</version>
              </dependency>
      
              <!-- spring aop织入包 -->
              <dependency>
                  <groupId>org.aspectj</groupId>
                  <artifactId>aspectjweaver</artifactId>
                  <version>1.9.5</version>
              </dependency>
      
              <!--mysql的驱动包-->
              <dependency>
                  <groupId>mysql</groupId>
                  <artifactId>mysql-connector-java</artifactId>
                  <version>5.1.47</version>
              </dependency>
      
              <!-- spring操作数据库的依赖包 -->
              <dependency>
                  <groupId>org.springframework</groupId>
                  <artifactId>spring-jdbc</artifactId>
                  <version>5.2.1.RELEASE</version>
              </dependency>
      
              <!--mybatis的依赖-->
              <dependency>
                  <groupId>org.mybatis</groupId>
                  <artifactId>mybatis</artifactId>
                  <version>3.5.2</version>
              </dependency>
      
              <!-- spring对于mybatis的整合 -->
              <dependency>
                  <groupId>org.mybatis</groupId>
                  <artifactId>mybatis-spring</artifactId>
                  <version>2.0.4</version>
              </dependency>
      ```
   
   2. 编写UserDao接口
   
      ```java
      public interface UserDao {
      
          public List<UserVo> selectUser();
      
          public int addUser(UserVo userVo);
      
          public int deleteUser(long id);
              
      }
      ```
   
   3. 编写mapper映射文件
   
      ```xml
      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE mapper
              PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
              "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
      
      <mapper namespace="com.chengchao001.dao.UserDao">
      
          <select id="selectUser" resultType="com.chengchao001.vo.UserVo">
              select
              USER_ID as userId,
              USER_NAME as userName,
              USER_AGE as userAge,
              USER_EMAIL as userEmail
           from vop_user_info_t;
          </select>
      
          <insert id="addUser" parameterType="com.chengchao001.vo.UserVo">
              insert into vop_user_info_t
              (
                USER_ID,USER_NAME,USER_AGE,USER_EMAIL
              ) values (
                #{userId},#{userName},#{userAge},#{userEmail}
              )
          </insert>
      
          <delete id="deleteUser" parameterType="long">
              delete from vop_user_info_t
              where USER_ID = #{id}
          </delete>
      </mapper>
      ```
   
   4. 编写UserDaoImpl实现类
   
      ```java
      public class UserDaoImpl implements UserDao {
      
          private SqlSessionTemplate sqlSessions;
          
          public void setSqlSessions(SqlSessionTemplate sqlSessions) {
              UserDao mapper = sqlSessions.getMapper(UserDao.class);
              this.sqlSessions = sqlSessions;
          }
      
          public List<UserVo> selectUser() {
              UserDao mapper = sqlSessions.getMapper(UserDao.class);
              return mapper.selectUser();
          }
      
          public int addUser(UserVo userVo) {
              UserDao mapper = sqlSessions.getMapper(UserDao.class);
              return mapper.addUser(userVo);
          }
      
          public int deleteUser(long id) {
              UserDao mapper = sqlSessions.getMapper(UserDao.class);
              return mapper.deleteUser(id);
          }
      }
      ```
   
   5. 编写Spring核心配置文件applicationContext.properties
   
      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
              https://www.springframework.org/schema/beans/spring-beans.xsd">
      
          <!--使用spring的数据源-->
          <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
              <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
              <property name="url" value="jdbc:mysql://localhost:3306/mydatabase?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8"/>
              <property name="username" value="root"/>
              <property name="password" value="686995"/>
          </bean>
      
          <!--使用spring—mybatis的sqlSessionFactory-->
          <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <property name="dataSource" ref="dataSource"/>
              <property name="mapperLocations" value="com/chengchao001/dao/UserDaoImpl.xml"/>
          </bean>
      
          <!--第一种方式使用SqlSessionTemplate，注入sqlSessionFactory-->
          <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
              <constructor-arg ref="sqlSessionFactory"/>
          </bean>
      
          <!--需要将SqlSessionTemplate的对象注入到dao的实现类中-->
          <bean id="userDao" class="com.chengchao001.dao.UserDaoImpl">
              <property name="sqlSessions" ref="sqlSession"/>
          </bean>
      
          <bean id="userService" class="com.chengchao001.service.impl.UserServiceImpl">
              <property name="userDao" ref="userDao"/>
          </bean>
      </beans>
      ```
   
   









