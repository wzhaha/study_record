# mybatis学习记录
#### 一. 配置
```
依赖注入
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>

mybatis配置文件
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>

类型别名（简化xml）
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
</typeAliases>
```
#### 二.多数据源配置
```
@Configuration
@MapperScan(basePackages = "com.example.demo", sqlSessionFactoryRef = "demoSqlSessionFactory")
public class DatasourceDemoConfig {

    @Bean(name = "demoDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.demo")
    public DataSource demoDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "demoSqlSessionFactory")
    public SqlSessionFactory demoSqlSessionFactory(@Qualifier("demoDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/*.xml"));
        //读取mybatis小配置文件
        // bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/test1/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "demoTransactionManager")
    public DataSourceTransactionManager demoTransactionManager(@Qualifier("demoDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "demoSqlSessionTemplate")
    public SqlSessionTemplate demoSqlSessionTemplate(@Qualifier("demoSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```
一个数据源对应一个SqlSessionFactory，连接n个数据库时需要手动建立多个SqlSessionFactory，用于多数据源中
#### 三.映射文件
namespace为指定的mapper interface
resultmap为指定的返回类型

##### sql配置顶级元素
- cache – 对给定命名空间的缓存配置。
- cache-ref – 对其他命名空间缓存配置的引用。
- resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
- ~~parameterMap – 已被废弃！老式风格的参数映射。更好的办法是使用内联参数，此元素可能在将来被移除。~~
- sql – 可被其他语句引用的可重用语句块。
- insert – 映射插入语句
- update – 映射更新语句
- delete – 映射删除语句
- select – 映射查询语句

##### 具体使用

+ select
```
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
相当于
SELECT * FROM PERSON WHERE ID=?
```
resultMap 和 resultType，不能同时使用，用于指定返回类型。

+ insert
    - 单行插入
    ```
    <insert id="insertAuthor">
      insert into Author (id,username,password,email,bio)
      values (#{id},#{username},#{password},#{email},#{bio})
    </insert>
    ```
    - 多行插入
    ```
    <insert id="insertAuthor" useGeneratedKeys="true"
        keyProperty="id">
      insert into Author (username, password, email, bio) values
      <foreach item="item" collection="list" separator=",">
        (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
      </foreach>
    </insert>
    ```

+ update
```
<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>
```

+ delete
```
<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```

+ sql
    - sql片段定义
      ```
      <sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
      ```
    - sql片段使用
      ```
      <select id="selectUsers" resultType="map">
        select
          <include refid="userColumns"><property name="alias" value="t1"/></include>,
          <include refid="userColumns"><property name="alias" value="t2"/></include>
        from some_table t1
          cross join some_table t2
      </select>
      ```
      
+ 参数
    指定传入参数的类型
    ```
    #{property,javaType=int,jdbcType=NUMERIC,numericScale=2}
    ```
    numericScale为小数点保留数位
    当参数可以为null的时候，必须指定jdbcType
    
+ 结果映射
    - resultMap定义
    ```
    <resultMap id="userResultMap" type="User">
      <id property="id" column="user_id" />
      <result property="username" column="user_name"/>
      <result property="password" column="hashed_password"/>
    </resultMap>
    ```
    - resultMap使用
    ```
    <select id="selectUsers" resultMap="userResultMap">
      select user_id, user_name, hashed_password
      from some_table
      where id = #{id}
    </select>
    ```
    - id & result
    ```
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    ```
    - 关联
    ```
    <select id="selectBlog" resultMap="blogResult">
      select
        B.id            as blog_id,
        B.title         as blog_title,
        B.author_id     as blog_author_id,
        A.id            as author_id,
        A.username      as author_username,
        A.password      as author_password,
        A.email         as author_email,
        A.bio           as author_bio
      from Blog B left outer join Author A on B.author_id = A.id
      where B.id = #{id}
    </select>
    <resultMap id="blogResult" type="Blog">
      <id property="id" column="blog_id" />
      <result property="title" column="blog_title"/>
      <association property="author" column="blog_author_id" javaType="Author" resultMap="authorResult"/>
    </resultMap>

    <resultMap id="authorResult" type="Author">
      <id property="id" column="author_id"/>
      <result property="username" column="author_username"/>
      <result property="password" column="author_password"/>
      <result property="email" column="author_email"/>
      <result property="bio" column="author_bio"/>
    </resultMap>
    ```
    - 集合(有多个关联)
    ```
    <resultMap id="blogResult" type="Blog">
      <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
    </resultMap>

    <select id="selectBlog" resultMap="blogResult">
      SELECT * FROM BLOG WHERE ID = #{id}
    </select>

    <select id="selectPostsForBlog" resultType="Post">
      SELECT * FROM POST WHERE BLOG_ID = #{id}
    </select>
    ```
    - 鉴别器
    ```
    <resultMap id="vehicleResult" type="Vehicle">
      <id property="id" column="id" />
      <result property="vin" column="vin"/>
      <result property="year" column="year"/>
      <result property="make" column="make"/>
      <result property="model" column="model"/>
      <result property="color" column="color"/>
      <discriminator javaType="int" column="vehicle_type">
        <case value="1" resultType="carResult">
          <result property="doorCount" column="door_count" />
        </case>
        <case value="2" resultType="truckResult">
          <result property="boxSize" column="box_size" />
          <result property="extendedCab" column="extended_cab" />
        </case>
        <case value="3" resultType="vanResult">
          <result property="powerSlidingDoor" column="power_sliding_door" />
        </case>
        <case value="4" resultType="suvResult">
          <result property="allWheelDrive" column="all_wheel_drive" />
        </case>
      </discriminator>
    </resultMap>
    ```
    相当于java中的switch case
 
+ 缓存 
    - cache
      缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
      ```
      <cache
        eviction="FIFO"
        flushInterval="60000"
        size="512"
        readOnly="true"/>
      ```
      eviction默认为LRU算法
    - cache-ref
    ```
    <cache-ref namespace="com.someone.application.data.SomeMapper"/>
    ```
    如果想要在多个命名空间中共享相同的缓存配置和实例，可以使用 cache-ref 元素来引用另一个缓存
