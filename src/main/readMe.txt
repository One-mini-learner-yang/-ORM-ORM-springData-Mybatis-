mybatis
    接口式编程，将接口交给xml（或注解，本文以xml为例说明）
springData
    接口继承JpaRepository<T(操作的实体类类型)，ID（操作的实体类主键类型）>和JpaSpecificationExecutor<T(操作的实体类类型)>
相关依赖（本文以springBoot，数据池以Druid作为例子说明）
    mybatis：mybatis-spring-boot-starter，druid
    springData：spring-boot-starter-data-jpa，druid
配置文件相关配置
    数据源相关配置
        spring.datasource.username=root
        spring.datasource.password=root
        spring.datasource.url=jdbc:mysql://127.0.0.1:3306/springBoot
        spring.datasource.driver-class-name=com.mysql.jdbc.Driver
        spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
        Druid相关配置
            spring.datasource.initialSize=5
            spring.datasource.minIdle=5
            spring.datasource.maxActive=20
            spring.datasource.maxWait=60000
            spring.datasource.timeBetweenEvictionRunsMillis=60000
            spring.datasource.minEvictableIdleTimeMillis=300000
            spring.datasource.validationQuery=SELECT 1 FROM DUAL
            spring.datasource.testWhileIdle=true
            spring.datasource.testOnBorrow=false
            spring.datasource.poolPreparedStatements=true
            #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
            spring.datasoure.filters=stat,wall,log4j
            spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
            spring.datasource.useGlobalDataSourceStat=true
            spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
    mybatis相关配置
        mybatis.mapper-locations="相关mapper。xml路径"
        注：此处也可进行缓存的开启配置，就不再此具体举例了
    springData相关配置
        spring.jpa.hibernate.ddl-auto=update
            有以下的属性
                  ddl-auto:create----每次运行该程序，没有表格会新建表格，表内有数据会清空
                  ddl-auto:create-drop----每次程序结束的时候会清空表
                  ddl-auto:update----每次运行程序，没有表格会新建表格，表内有数据不会清空，只会更新
                  ddl-auto:validate----运行程序会校验数据与数据库的字段类型是否相同，不同会报错
        spring.jpa.show-sql=true
数据的操作
    mybatis
        1.正常写sql语句
        2.动态拼接sql
            if标签：条件判断下的拼接
            where标签：代替sql语句的where
                   <select id="findActiveBlogWithTitleLike"
                        resultType="Blog">
                     SELECT * FROM BLOG
                     WHERE state = ‘ACTIVE’
                     <if test="title != null">
                       AND title like #{title}
                     </if>
                   </select>
            choose标签：多个条件下的选择拼接
             <select id="findActiveBlogLike"
                      resultType="Blog">
                   SELECT * FROM BLOG WHERE state = ‘ACTIVE’
                   <choose>
                     <when test="title != null">
                       AND title like #{title}
                     </when>
                     <when test="author != null and author.name != null">
                       AND author_name like #{author.name}
                     </when>
                     <otherwise>
                       AND featured = 1
                     </otherwise>
                   </choose>
             </select>
             trim标签：给sql语句段加前缀，后缀，prefix，suffix分别为前缀，后缀，prefixOverrides，suffixOverrides为将sql语句段首部，尾部去掉
                <select id="dynamicTrimTest" parameterType="Blog" resultType="Blog">
                    select * from t_blog
                    <trim prefix="where" prefixOverrides="and |or">
                        <if test="title != null">
                            title = #{title}
                        </if>
                        <if test="content != null">
                            and content = #{content}
                        </if>
                        <if test="owner != null">
                            or owner = #{owner}
                        </if>
                    </trim>
                </select>
             set标签：代替sql语句的set
              update Author
                     <set>
                       <if test="username != null">username=#{username},</if>
                       <if test="password != null">password=#{password},</if>
                       <if test="email != null">email=#{email},</if>
                       <if test="bio != null">bio=#{bio}</if>
                     </set>
                   where id=#{id}
              foreach标签：sql语句的循环动态插入标签
              item参数：在循环过程中，给循环的元素起的别名
              index参数：指定一个名字，用于表示在迭代过程中，每次迭代到的位置
              open参数：循环拼接语句以什么开始
              close参数：循环拼接语句以什么结束
              collection：当循环对象为list时，该参数值为list，当循环对象为array时，参数值为array，当循环对象为map时，参数值为循环对象对应的key值
              SELECT *
                    FROM POST P
                    WHERE ID in
                    <foreach item="item" index="index" collection="list"
                        open="(" separator="," close=")">
                          #{item}
                    </foreach>
    springData
        springData是基于Jpa的一个框架，使用DAO层要建立相应调用实体类与对应表的关系
        例如：
            @Table(name="JPA_ORDERS")
            @Entity
            public class Order {

                @GeneratedValue(strategy=GenerationType.IDENTITY)
                @Id
                private Integer id;

                @Column(name="ORDER_NAME")
                private String orderName;
            }
        @Table：声明该实体类与表的关系，参数表明
        @Entity：表明该类为一个实体类
        @GeneratedValue：自动生成策略（针对于主键），参数strategy为生成策略
        strategy的值为以下可能
        1.GenerationType.IDENTITY：对于支持主键自增长的数据库（如mysql），主键采用主键自增长
        2.GenerationType.SEQUENCE：对于不支持主键自增长的数据库（如oracle），主键采用序列的方式
        3.GenerationType.TABLE：采用一个外加表表来实现主键的自增，经常配合@TableGenerator（指定生成序列表）使用
        例如：
                @Id
                @GeneratedValue(strategy = GenerationType.TABLE, generator = "roleSeq")
                @TableGenerator(name = "roleSeq", allocationSize = 1, table = "seq_table", pkColumnName = "seq_id", valueColumnName = "seq_count")
                private Integer id;
                @GeneratedValue的属性generator指定生成策略的名，@TableGenerator指定生成相应的序列表，其属性allocationSize为增长的步长，pkColumnName为该主键生成策略对应键值的名称
                valueColumnName为该主键当前的值，会不断根据步长自动增加
        若不使用@TableGenerator的话，Jpa会自动生成一个名为sequence的表(SEQ_NAME,SEQ_COUNT)，该表一般为2个字段，一个为改生成策略的名称，另一个为该表的最大序号，会不断累加
        4.GenerationType.AUTO：Jpa会自动根据数据库选择以上三种方式，该做法也是比较常用的方法
        @Column：说明该属性与哪个字段对应
        实现数据访问的方法
            1.使用springData封装的方法
                接口JpaRepository封装了一些方法
                    findAll()：查询全部
                    findOne()：查询一条，参数主键值
                    save()：保存一条数据
                    delete()：删除一条数据
                    count()：数据的数量
                    exists()：判断数据是否存在，参数主键值
            2.利用自定义方法命名规则查询
                在DAO接口处（该接口继承JpaRepository<T(操作的实体类类型)，ID（操作的实体类主键类型）>和JpaSpecificationExecutor<T(操作的实体类类型)>）按照命名规则自定义方法
                命名规则：
                    1.单一条件：findBy +属性名（实体类中的属性名），例如：public User findById(Long id);
                               findBy +属性名 +like（模糊查询）,例如：public User findByUserNameLike（String userName）;
                    2.多个条件：findBy +属性名 +查询方式（若为精确查询，此处不写，若为模糊查询，此处写like） +多条件连接符（and/or） +属性名 +查询方式
                               例如：public User findByIdAndUsernameLike(Long id,String userName);
            3.自定义jpql/sql
            jpql：是Jpa针对于对象操作的数据库查询语言，类似sql，区别是jpql针对于实体对象，而sql针对于数据库的对应字段
            本文以查询和更新举例二者差距
            查询：
                jpql：from User where userName=?
                sql：select * from c_user where c_userName=?
            更新：
                jpql：Update User set userName=? where userId=?
                sql：update c_user set c_userName=? where c_userId=?
            在DAO接口定义的方法前使用@Query注解
            @Query的属性：
                value：jpql/sql语句
                nativeQuery：为true则当前以sql方式及进行操作，为false则当前以jpql方式进行操作，默认为false
            占位符的赋值
                在问号后直接写对应方法参数的位置即可
                例如：
                    @Query(value="Update User set userName=?1 where userId=?2")
                    public void update(String userName,Long id);
            4.动态拼接sql



