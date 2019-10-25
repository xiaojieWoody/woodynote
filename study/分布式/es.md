![image-20191014222023687](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014222023687.png)

![image-20191014222044409](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014222044409.png)





* head插件
  * npm install
  * npm run start
  * http://localhost:9100 
* cd elasticsearch..
  * vim config/elasticsearch.yml
  * http.cors.enabled: true
    http.cors.allow-origin: "*"
  * ./bin/elasticsearch -d
* cd elasticsearch-head-master
  * npm run start
  * http://localhost:9100/



* 分布式扩容

  * 新终端

  * vim config/elasticsearch.yml

    * cluster.name: woody
      node.name: master
      node.master: true

      network.host: 127.0.0.1

  *  ps -ef|grep `pwd`

    * kill 7380

  * 重新启动es和head插件

  * http://localhost:9100/

  * http://localhost:9200/

  * 复制两份slave目录，解压

    * ![image-20191014231350047](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014231350047.png)

    * cluster.name: woody
      node.name: slave1

      network.host: 127.0.0.1
      http.port: 8200
      discovery.zen.ping.unicast.hosts: ["127.0.0.1"]

    *  ./bin/elasticsearch -d

    * ![image-20191014231842411](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014231842411.png)

    * cluster.name: woody
      node.name: slave2

      network.host: 127.0.0.1
      http.port: 8000
      discovery.zen.ping.unicast.hosts: ["127.0.0.1"]

    * ![image-20191014232124444](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014232124444.png)





![image-20191014232423726](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014232423726.png)



![image-20191014232349901](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014232349901.png)



索引——database

类型——table

文档——一行记录



![image-20191014232711112](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014232711112.png)







![image-20191014232852118](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014232852118.png)



### 索引

* http://localhost:9100/
* 创建索引，小写 book
* 分类-索引信息-mappings为空-非结构化索引
  * 结构化索引
  * 非结构化索引

* 创建结构化索引

  * ![image-20191014234656982](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014234656982.png)

* ![image-20191014235738793](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191014235738793.png)

* ```json
  {
  	"settings":{
  		"number_of_shards":3,
  		"number_of_replicas":1
  	},
  	"mappings":{
  		"man":{
  			"properties":{
  				"name":{
  					"type":"text"
  				},
  				"country":{
  					"type":"keyword"
  				},
  				"age":{
  					"type":"integer"
  				},
  				"date":{
  					"type":"date",
  					"format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
  				}
  			}
  		},
  		"woman":{
  			
  		}
  	}
  }
  ```

* 



### 插入

* 指定文档id插入
* 自动产生文档id插入
* ![image-20191015000008540](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015000008540.png)
* ![image-20191015000649621](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015000649621.png)
* ![image-20191015000704898](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015000704898.png)
* ![image-20191015000912408](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015000912408.png)
* ![image-20191015000942895](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015000942895.png)





### 修改

* 直接修改文档
* 脚本修改文档
* ![image-20191015001539667](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015001539667.png)
* ![image-20191015001718691](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015001718691.png)
* ![image-20191015001834476](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015001834476.png)
* 



### 删除

* 删除文档
* 删除索引
* ![image-20191015002215493](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015002215493.png)
* ![image-20191015002246865](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015002246865.png)



### 查询

* 简单查询
* 条件查询
* 聚合查询

![image-20191015002753647](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015002753647.png)

![image-20191015002939373](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015002939373.png)

![image-20191015002922480](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015002922480.png)

![image-20191015003132500](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003132500.png)

 ![image-20191015003314628](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003314628.png)

![image-20191015003525558](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003525558.png)

![image-20191015003653253](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003653253.png)

![image-20191015003729690](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003729690.png)





## 高级查询

 ![image-20191015003818868](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003818868.png)

![image-20191015003837624](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003837624.png)

![image-20191015003908437](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015003908437.png)

![image-20191015004010380](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004010380.png)

![image-20191015004127247](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004127247.png)

![image-20191015004229814](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004229814.png)

![image-20191015004403944](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004403944.png)



![image-20191015004459970](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004459970.png)

![image-20191015004558429](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004558429.png)

![image-20191015004647881](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004647881.png)





 ![image-20191015004739745](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004739745.png)

![image-20191015004856042](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004856042.png)

![image-20191015004934133](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015004934133.png)

![image-20191015005122367](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015005122367.png)

![image-20191015005306111](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015005306111.png)

![image-20191015005402219](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015005402219.png)

![image-20191015005515933](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015005515933.png)







## SpringBoot + ES

 ![image-20191015005905400](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015005905400.png)



![image-20191015010120370](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015010120370.png)



![image-20191015010600348](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015010600348.png)

![image-20191015010732325](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015010732325.png)

![image-20191015010925852](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015010925852.png)





![image-20191015011546417](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015011546417.png)

![image-20191015011609546](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015011609546.png)







![image-20191015011846518](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015011846518.png)

![image-20191015012030401](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015012030401.png)

![image-20191015012151952](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191015012151952.png)

```java
WildcardQueryBuilder wBuilder = QueryBuilders.wildcardQuery("c_name", queryBean.getResName());
        SearchResponse response = esClient.prepareSearch(ESConfig.RES_EXPORT_INDEX).setTypes(ESConfig.RES_EXPORT_TYPE)
                .setQuery(wBuilder)
//                .setFrom(pageNo).setSize(pageSize).setExplain(true)
                .execute()
                .actionGet();


String id = "qnUROWcBmnqHZhEARIjE";
GetResponse response1 = esClient.prepareGet(bloodIndex, resType, id).get();
if(!response1.isExists()) {
  log.error("........资源不存在.........");
  return new ResponseResult(200, "", "succeess", null);
}
log.info("--------" + JSON.toJSONString(response1.getSource()) + "----------");
return new ResponseResult(200, "", "succeess", response1.getSource());


//        String id = "TABLE.b38c50b200254e43850d0225f88285cc.null.test_date";
//        GetResponse response = esClient.prepareGet(ESConfig.RES_EXPORT_INDEX, ESConfig.RES_EXPORT_TYPE, id).get();
//        if(!response.isExists()) {
//            log.error("........资源不存在.........");
//            return new ResponseResult(200, "", "succeess", null);
//        }
//        log.info("--------" + JSON.toJSONString(response.getSource()) + "----------");
//        return new ResponseResult(200, "", "succeess", response.getSource());


String cId = "DS.a9602f28f81343acbde4676fdc2bb2b8";
WildcardQueryBuilder wBuilder = QueryBuilders.wildcardQuery("c_id", cId);
SearchResponse response = esClient.prepareSearch(resTreeIndex).setTypes(resType)
  .setQuery(wBuilder)
  .execute()
  .actionGet();
SearchHits hits = response.getHits();
log.info("--------" + JSON.toJSONString(hits.getHits()) + "----------");
//        return new ResponseResult(200, "", "succeess", response1.getSource());


//@Slf4j
//@Configuration
public class ESClient {

//    @Value("${smart.config.elasticsearch.host}")
//    private String host;
//
//    @Value("${smart.config.elasticsearch.port}")
//    private Integer port;
//
//    @Bean
//    public TransportClient client() throws UnknownHostException {
//
//        Settings settings = Settings.builder().put("client.transport.sniff", true).build();
//        System.setProperty("es.set.netty.runtime.available.processors", "false");
//        TransportClient client = new PreBuiltTransportClient(settings).addTransportAddress(
//                new TransportAddress(InetAddress.getByName(host), port));
//        return client;
//    }
}
```



