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

