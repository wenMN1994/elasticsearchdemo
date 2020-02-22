# Elasticsearch 入门
## 全文检索工具elasticsearch

### Elasticsearch 安装

1. lucene与elasticsearch
   	咱们之前讲的处理分词，构建倒排索引，等等，都是这个叫lucene的做的。那么能不能说这个lucene就是搜索引擎呢？
      	还不能。lucene只是一个提供全文搜索功能类库的核心工具包，而真正使用它还需要一个完善的服务框架搭建起来的应用。
      	好比lucene是类似于jdk，而搜索引擎软件就是tomcat 的。
      	目前市面上流行的搜索引擎软件，主流的就两款，elasticsearch和solr,这两款都是基于lucene的搭建的，可以独立部署启动的搜索引擎服务软件。由于内核相同，所以两者除了服务器安装、部署、管理、集群以外，对于数据的操作，修改、添加、保存、查询等等都十分类似。就好像都是支持sql语言的两种数据库软件。只要学会其中一个另一个很容易上手。
      	从实际企业使用情况来看，elasticSearch的市场份额逐步在取代solr，国内百度、京东、新浪都是基于elasticSearch实现的搜索功能。国外就更多了 像维基百科、GitHub、Stack Overflow等等也都是基于ES的

2. elasticSearch的使用场景

   - 为用户提供按关键字查询的全文搜索功能。

   - 著名的ELK框架(ElasticSearch,Logstash,Kibana)，实现企业海量日志的处理分析的解决方案。大数据领域的重要一份子。

3. elasticsearch的基本概念

   | cluster  | 整个elasticsearch 默认就是集群状态，整个集群是一份完整、互备的数据。 |
   | -------- | ------------------------------------------------------------ |
   | node     | 集群中的一个节点，一般只一个进程就是一个node                 |
   | shard    | 分片，即使是一个节点中的数据也会通过hash算法，分成多个片存放，默认是5片。 |
   | index    | 相当于rdbms的database, 对于用户来说是一个逻辑数据库，虽然物理上会被分多个shard存放，也可能存放在多个node中。 |
   | type     | 类似于rdbms的table，但是与其说像table，其实更像面向对象中的class , 同一Json的格式的数据集合。 |
   | document | 类似于rdbms的 row、面向对象里的object                        |
   | field    | 相当于字段、属性                                             |

   

4. 利用kibana学习 elasticsearch restful api (DSL)

   - es中保存的数据结构

     ```
     public class  Movie {
     	 String id;
          String name;
          Double doubanScore;
          List<Actor> actorList;
     }
     
     public class Actor{
         String id;
         String name;
     }
     ```

     这两个对象如果放在关系型数据库保存，会被拆成2张表，但是elasticsearch是用一个json来表示一个document。

     所以他保存到es中应该是：

     ```
     {
       “id”:”1”,
       “name”:”operation red sea”,
       “doubanScore”:”8.5”,
       “actorList”:[  
         {“id”:”1”,”name”:”zhangyi”},
         {“id”:”2”,”name”:”haiqing”},
         {“id”:”3”,”name”:”zhanghanyu”}
        ]
     }
     
     ```

   - 对数据的操作  

     - 查看es中有哪些索引  

       ```
       GET /_cat/indices?v
       ```

       es 中会默认存在一个名为.kibana的索引

       表头的含义

       | health         | green(集群完整) yellow(单点正常、集群不完整) red(单点不正常) |
       | -------------- | ------------------------------------------------------------ |
       | status         | 是否能使用                                                   |
       | index          | 索引名                                                       |
       | uuid           | 索引统一编号                                                 |
       | pri            | 主节点几个                                                   |
       | rep            | 从节点几个                                                   |
       | docs.count     | 文档数                                                       |
       | docs.deleted   | 文档被删了多少                                               |
       | store.size     | 整体占空间大小                                               |
       | pri.store.size | 主节点占                                                     |

     - 增加一个索引  

       ```
       PUT /movie_index
       ```

     - 删除一个索引

       ```
       ES 是不删除也不修改任何数据 DELETE /movie_index
       ```

     - 新增文档 

       格式 PUT /index/type/id

       ```
       PUT /movie_index/movie/1
       { "id":1,
         "name":"operation red sea",
         "doubanScore":8.5,
         "actorList":[  
           {"id":1,"name":"zhang yi"},
           {"id":2,"name":"hai qing"},
           {"id":3,"name":"zhang han yu"}
         ]
       }
       
       PUT /movie_index/movie/2
       {
         "id":2,
         "name":"operation meigong river",
         "doubanScore":8.0,
         "actorList":[  
       	{"id":3,"name":"zhang han yu"}
         ]
       }
       
       PUT /movie_index/movie/3
       {
         "id":3,
         "name":"incident red sea",
         "doubanScore":5.0,
         "actorList":[  
       	{"id":4,"name":"zhang chen"}
         ]
       }
       ```

       如果之前没建过index或者type，es 会自动创建。

     - 直接用id查找  

       ```
      GET movie_index/movie/1
       ```

     - 修改—整体替换  
   
       ```
       PUT /movie_index/movie/3
       {
         "id":"3",
         "name":"incident red sea",
         "doubanScore":"5.0",
         "actorList":[  
       	{"id":"1","name":"zhang chen"}
         ]
       }
       ```
   
     - 修改—某个字段  
   
       ```
       POST movie_index/movie/3/_update
       { 
         "doc": {
           "doubanScore":"7.0"
         } 
       }
       ```
   
     - 删除一个document  
   
       ```
       DELETE movie_index/movie/3
       ```
   
     - 搜索type全部数据 
   
       ```
       GET movie_index/movie/_search
       ```
   
       结果
   
       ```
       {
         "took": 2,    //耗费时间 毫秒
         "timed_out": false, //是否超时
         "_shards": {
           "total": 5,   //发送给全部5个分片
           "successful": 5,
           "skipped": 0,
           "failed": 0
         },
         "hits": {
           "total": 3,  //命中3条数据
           "max_score": 1,   //最大评分
           "hits": [  // 结果
             {
               "_index": "movie_index",
               "_type": "movie",
               "_id": 2,
               "_score": 1,
               "_source": {
                 "id": "2",
                 "name": "operation meigong river",
                 "doubanScore": 8.0,
                 "actorList": [
                   {
                     "id": "1",
                     "name": "zhang han yu"
                   }
                 ]
               }
                 。。。。。。。。
                 。。。。。。。。
             }
       
       ```
   
     - 按条件查询(全部)
   
       ```
       GET movie_index/movie/_search
       {
         "query":{
           "match_all": {}
         }
       }
       ```
   
     - 按分词查询
   
       ```
       GET movie_index/movie/_search
       {
         "query":{
           "match": {"name":"red"}
         }
       }
       注意结果的评分
       ```
   
     - 按分词子属性查询
   
       ```
       GET movie_index/movie/_search
       {
         "query":{
           "match": {"actorList.name":"zhang"}
         }
       }
       ```
   
     - #### match phrase
   
       ```
       GET movie_index/movie/_search
       {
           "query":{
             "match_phrase": {"name":"operation red"}
           }
       }
       按短语查询，不再利用分词技术，直接用短语在原始数据中匹配
       ```
   
     - fuzzy查询
   
       ```
       GET movie_index/movie/_search
       {
           "query":{
             "fuzzy": {"name":"rad"}
           }
       }
       校正匹配分词，当一个单词都无法准确匹配，es通过一种算法对非常接近的单词也给与一定的评分，能够查询出来，但是消耗更多的性能。
       ```
   
     - 过滤--查询后过滤
   
       ```
       GET movie_index/movie/_search
       {
           "query":{
             "match": {"name":"red"}
           },
           "post_filter":{
             "term": {
               "actorList.id": 3
             }
           }
       }
       ```
   
     - 过滤--查询前过滤（推荐）
   
       ```
       GET movie_index/movie/_search
       { 
           "query":{
               "bool":{
                 "filter":[ {"term": {  "actorList.id": "1"  }},
                            {"term": {  "actorList.id": "3"  }}
                  ], 
                  "must":{"match":{"name":"red"}}
                }
           }
       }
       ```
   
     - 过滤--按范围过滤
   
       ```
       GET movie_index/movie/_search
       {
          "query": {
            "bool": {
              "filter": {
                "range": {
                   "doubanScore": {"gte": 8}
                }
              }
            }
          }
       }
       ```
   
       关于范围操作符：
   
       | gt   | 大于     |
       | ---- | -------- |
       | lt   | 小于     |
       | get  | 大于等于 |
       | lte  | 小于等于 |
   
     - 排序
   
       ```
       GET movie_index/movie/_search
       {
         "query":{
           "match": {"name":"red sea"}
         }
         , "sort": [
           {
             "doubanScore": {
               "order": "desc"
             }
           }
         ]
       }
       ```
   
     - 分页查询
   
       ```
       GET movie_index/movie/_search
       {
         "query": { "match_all": {} },
         "from": 1,
         "size": 1
       }
       ```
   
     - 指定查询的字段
   
       ```
       GET movie_index/movie/_search
       {
         "query": { "match_all": {} },
         "_source": ["name", "doubanScore"]
       }
       ```
   
     - 高亮
   
       ```
       GET movie_index/movie/_search
       {
           "query":{
             "match": {"name":"red sea"}
           },
           "highlight": {
             "fields": {"name":{} }
           }
           
       }
       
       
       
       自定义标签
       GET movie_index/movie/_search
       {
           "query":{
             "match": {"name":"red sea"}
           },
           "highlight": {
             "pre_tags": ["<b>"], 
             "post_tags": ["</b>"], 
             "fields": {"name":{} }
           }
           
       }
       ```
   
     - 聚合
   
       ```
       取出每个演员共参演了多少部电影
       GET movie_index/movie/_search
       { 
         "aggs": {
           "groupby_actor": {
             "terms": {
               "field": "actorList.name.keyword"  
             }
           } 
         }
       }
       
       每个演员参演电影的平均分是多少，并按评分排序
       GET movie_index/movie/_search
       { 
         "aggs": {
           "groupby_actor_id": {
             "terms": {
               "field": "actorList.name.keyword" ,
               "order": {
                 "avg_score": "desc"
                 }
             },
             "aggs": {
               "avg_score":{
                 "avg": {
                   "field": "doubanScore" 
                 }
               }
              }
           } 
         }
       }
       ```
   
       
   
     - 
   
   - 
   
   - 

