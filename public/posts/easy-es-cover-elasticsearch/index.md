# 用easy-Es简化ElasticSearch操作


&lt;!--more--&gt;

## 一、前言

ElasticSearch的Java客户端中，[spring-data-elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch) 简化了 增删改、建索引等，没有简化 复杂查询 编码。[easy-es](https://www.easy-es.cn/) API 类似 Mybatis-Plus，大幅降低开发门槛，减少代码量，支持 自定义排序、权重、原生查询，留下了 广阔的调整空间

## 二、Java客户端

- ~~[Java Transport Client](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/index.html)~~：官方已弃用，二进制协议，版本强绑定，不建议使用
- ~~[Jest](https://github.com/searchbox-io/Jest)~~：4年多没发布新版本，不维护了，不建议使用
- [Java REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)：7.15 以下版本适用
- [Elasticsearch Java API Client](https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html)：7.16 版本发布测试版，建议es 8.0 以上使用
- [spring-data-elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch)：简化了 增删改、建索引等，没有简化 复杂查询 编码
- [easy-es](https://www.easy-es.cn/)：API 类似 Mybatis-Plus，大幅降低开发门槛，减少代码量，支持 自定义排序、权重、原生查询，留下了 广阔的调整空间

## 三、easy-es

1. [MySQL和 easy-es 语法对比 | Easy-Es](https://www.easy-es.cn/pages/8f3438/)
   
   | MySQL          | Easy-Es | es-DSL/es java api                        |
   | -------------- | ------- | ----------------------------------------- |
   | and            | and     | must                                      |
   | or             | or      | should                                    |
   | =              | eq      | term                                      |
   | !=             | ne      | boolQueryBuilder.mustNot(queryBuilder)    |
   | &gt;              | gt      | QueryBuilders.rangeQuery(&#39;es field&#39;).gt() |
   | &gt;=             | ge      | .rangeQuery(&#39;es field&#39;).gte()             |
   | &lt;              | lt      | .rangeQuery(&#39;es field&#39;).lt()              |
   | &lt;=             | le      | .rangeQuery(&#39;es field&#39;).lte()             |
   | like &#39;%field%&#39; | like    | QueryBuilders.wildcardQuery(field,value)  |
   | ...            |         |                                           |

2. [spring-boot配置](https://www.easy-es.cn/pages/eddebb/)
   
   注意：**`adress 不能以 http(s):// 开头`**
   
   在 application.yml 配置文件中添加：
   
   ```yml
   easy-es:
     enable: true # 是否开启EE自动配置 默认开启,可缺省
     address: 域名:9200
     username: elastic
     password: 密码
     banner: false # 默认为true 打印banner 若您不期望打印banner,可配置为false
     global-config:
       process-index-mode: smoothly #索引处理模式,smoothly:平滑模式,默认开启此模式, not_smoothly:非平滑模式, manual:手动模式
       print-dsl: false # 开启控制台打印通过本框架生成的DSL语句，默认为开启，测试稳定后的生产环境建议关闭，以提升少量性能
       async-process-index-blocking: false # 异步处理索引是否阻塞主线程 默认阻塞 数据量过大时调整为非阻塞异步进行 项目启动更快
       db-config:
         table-prefix: # 索引前缀,可用于区分环境 默认为空 用法和MP的tablePrefix一样的作用和用法
         field-strategy: not_null # 字段更新策略 默认为not_null
         refresh-policy: none # 默认为不刷新，wait_until对写入性能影响也很大
   ```

3. [添加依赖](https://www.easy-es.cn/pages/04414d/)
   
   如 maven，以 1.1.1 版本为例(最新版本是 2.0.0-beta1，后续讨论)：
   
   ```xml
   &lt;dependency&gt;
     &lt;groupId&gt;cn.easy-es&lt;/groupId&gt;
     &lt;artifactId&gt;easy-es-boot-starter&lt;/artifactId&gt;
     &lt;version&gt;1.1.1&lt;/version&gt;
   &lt;/dependency&gt;
   ```

4. [编写实体类](https://www.easy-es.cn/pages/ac41f0/)
   
   索引名称：默认是 类名转全小写(没有驼峰了)，建议驼峰类名指定
   
   keepGlobalPrefix：配合全局配置 easy-es.global-config.db-config.table-prefix 实现多个环境公用es
   
   String类型字段，1.1.1 版本默认为 keyword，为了便于后续升级，建议指定
   
   ```java
   @Data
   @IndexName(value = &#34;document&#34;, keepGlobalPrefix = true)
   public class Document {
       /**
        * es中的id
        */
       private String id;
   
       /**
        * 文档标题
       */
       @IndexField(fieldType = FieldType.KEYWORD)
       private String title;
   
       /**
        * 文档内容
        */
       @IndexField(fieldType = FieldType.TEXT, analyzer = Analyzer.IK_SMART)
       private String content;
   }
   ```

5. 编写Mapper类
   
   ```java
   public interface DocumentMapper extends BaseEsMapper&lt;Document&gt; {
   }
   ```

6. 添加 @EsMapperScan 注解
   
   ```java
   @SpringBootApplication
   @EsMapperScan(&#34;com.xpc.easyes.sample.mapper&#34;)
   public class Application {
   
       public static void main(String[] args) {
           SpringApplication.run(Application.class, args);
       }
   
   }
   ```

7. 保存
   
   ```java
   Document document = new Document();
   document.setTitle(&#34;老汉&#34;);
   document.setContent(&#34;推*技术过硬&#34;);
   int successCount = documentMapper.insert(document);
   ```

8. 更新
   
   ```java
   Document document1 = new Document();
   document1.setId(&#34;id&#34;);
   document1.setTitle(&#34;title1&#34;);
   documentMapper.updateById(document1);
   ```

9. 分页查询
   
   ```java
   documentMapper.pageQuery(EsWrappers.lambdaQuery(Document.class).eq(Document::getTitle, &#34;老汉&#34;), 1, 20)
   ```

## 四、总结

[easy-es](https://www.easy-es.cn/)，大幅降低开发门槛，减少代码量，支持 自定义排序、权重、原生查询，留下了 广阔的调整空间


![Gopher毛](https://blog.huchao.vip/picx-images-hosting/blog/%E5%85%AC%E4%BC%97%E5%8F%B7.58h8ppzfbd.webp)


---

> 作者: [胡超](https://github.com/mao888)  
> URL: http://localhost:1313/posts/easy-es-cover-elasticsearch/  
> 转载 URL: https://www.890808.xyz/easy-es-cover-elasticsearch/
