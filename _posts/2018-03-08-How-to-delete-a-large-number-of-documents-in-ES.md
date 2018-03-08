---
layout: post
date: 2018-03-08 16:40
title: How to delete a large number of documents in ES
description:  es删除大量文档.
categories: [ElasticSearch]
tags: [ElasticSearch,JestClient]
---

```
年后归来，进入客户后台查看定时任务执行的spark数据是否正常，点击进入后不堪入目，原本一个接口的图表只有10条数据，
结果涌现了数百条，且这些数据非常的有规律，相识度非常高，初步判断，“相同数据，复制了30+份！”。该报表为月报，在每
月1号3时后台spark从es读出数据，然后写回es，再供前端查询es。查看crontab脚本后，发现误将“每月1号3时”(0 0 3 1 *)
写成“每天3时”(0 3 * * *)执行...ops.开通该业务的公司有20个左右，总的数据冗余量在50000-100000条左右。
```

# es删除方式
平时开发日常用，一般设计的es操作为create，query或者基于aggs上的query，几乎没有任何DELETE的业务接口；因query能通过range筛选出符合一定日期范围的数据，类似的思维，DELETE命令是否也能圈定日期内的数据，批量删除呢？

并没有！！！
只有
```python
DELETE /index/type/id
```

数万条数据...WTF？？？

产品已经上线，es服务器ip访问不了，但spark服务器环境可通过内网ip访问es....so，只能另起一个小项目通过es java api先查询数这些数据的id，然后放入集合，再遍历一条一条执行DELETE，写好测试通过后，将该项目打包成jar包，在spark环境运行，通过浏览器调用暴露的restful接口传递具体的企业Id、时间范围等条件执行该删除接口。

这边并没有使用bulk批量删除....

# java获取所有冗余文档的ID遍历调用删除接口
通过spring boot另起项目，很快就搭建好了运行环境，接下来就是开发mvc的restful风格的接口去循环调用删除命令了。

## jar依赖
```java
<dependency>
			<groupId>org.elasticsearch</groupId>
			<artifactId>elasticsearch</artifactId>
			<version>5.1.1</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>io.searchbox</groupId>
			<artifactId>jest</artifactId> //推荐使用jest，elasticsearch-rest-high-level-client好难用
			<version>2.4.0</version>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.32</version>
		</dependency>
	</dependencies>

```

## es配置
application.properties配置
```java
spring.elasticsearch.jest.uris=http://10.10.53.187:9200/,http://10.10.52.143:9200/,http://10.10.54.45:9200/
spring.elasticsearch.jest.connection-timeout=1500
spring.elasticsearch.jest.read-timeout=10000

```

java配置文件
```java
package com.tinet.config;

import com.google.gson.GsonBuilder;
import io.searchbox.client.JestClient;
import io.searchbox.client.JestClientFactory;
import io.searchbox.client.config.HttpClientConfig;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.ArrayList;
import java.util.List;

@Configuration
public class ElasticsearchConfiguration {
	
	@Value("${spring.elasticsearch.jest.uris}")
	private String serverUri;
	@Value("${spring.elasticsearch.jest.connection-timeout}")
	private Integer connTimeout;
	@Value("${spring.elasticsearch.jest.read-timeout}")
	private Integer readTimeout;
	
	@Bean
	public JestClient getJestClient() {
		String[] serverUris=serverUri.split(","); //集群，多个地址
		List<String> serverUriList=new ArrayList<>();
		if (serverUris.length > 1){
			serverUri=serverUris[0];
			for (int i=1;i<serverUris.length;i++){
				serverUriList.add(serverUris[i]);
			}
		}
		JestClientFactory factory = new JestClientFactory();
		factory.setHttpClientConfig(new HttpClientConfig.Builder(serverUri).addServer(serverUriList)
				.gson(new GsonBuilder().setDateFormat("yyyy-MM-dd'T'hh:mm:ss").create()).connTimeout(connTimeout)
				.readTimeout(readTimeout).multiThreaded(true).build());
		return factory.getObject();
	}
	
	@Bean
	public SearchSourceBuilder getSearchSourceBuilder() {
		return SearchSourceBuilder.searchSource();
	}
}
```
## 需求代码实现
主要为获取query后的document ID.
```java
try {
            searchResult = super.jestClient.execute(search);
        } catch (IOException e) {
            logger.error("查询ES异常", e);
        }

        if (searchResult == null || !searchResult.isSucceeded()) {
            logger.error(searchResult.getErrorMessage());
        }

        List<String> dataList = null;
        if(searchResult.isSucceeded()){
            String jsonString = searchResult.getJsonString();
            logger.debug(jsonString);
            // 得到主通道数据
            JSONObject fastResult = JSON.parseObject(jsonString);
            JSONObject fastHits = fastResult.getJSONObject("hits");
            JSONArray fastArray = fastHits.getJSONArray("hits");


            // 处理主通道数据
            dataList = new ArrayList<>();
            for (int i = 0; i < fastArray.size(); i++) {
                JSONObject one = fastArray.getJSONObject(i);
                JSONObject fastSource = one.getJSONObject("_source");

                dataList.add(one.getString("_id"));
                System.out.println(one.getString("_id"));



                Delete delete = new Delete.Builder(one.getString("_id")).index(generateSparkIndex(date, Const.ELASTICSERACH_ALIAS)).type("spark").build();

                JestResult result = null ;
                try {
                    result = jestClient.execute(delete);
                    logger.info("deleteDocument == " + result.getJsonString());
                    Thread.sleep(50); //适当控制下删除速度
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
```
## 获取一个月的索引
该项目部分索引采用一天一个索引的方式，而不是一月一个，因此在查询一个月的时候，需自己加工下索引名字，顺便记录下:
用户传入开始时间和结束时间，格式为yyyy-MM-dd,最后获取单个索引的格式为cdr_yyyyMMdd_alias,一个月的则【cdr_yyyyMMdd_alias，cdr_yyyyMMdd_alias....】
```java
 protected static final String CDR_INDEX_PARSE_PATTERN = "yyyy-MM-dd";
 private static final String CDR_INDEX_FORMAT_PATTERN = "yyyyMMdd";
 private static final String CDR_INDEX_PREFIX = "cdr_";
 
 /**
     * 获取查询使用的ES索引集合
     *
     * @param startTime 起始时间
     * @param endTime   结束时间
     * @return 索引集合，例如：
     * <p>
     * [cdr_20170601_alias, cdr_20170602_alias]
     * </p>
     */
    protected static List<String> generateIndexList(String startTime, String endTime, String esAlias) {


        LocalDate startDate = LocalDate.parse(startTime, DateTimeFormatter.ofPattern(CDR_INDEX_PARSE_PATTERN));
        LocalDate endDate = LocalDate.parse(endTime, DateTimeFormatter.ofPattern(CDR_INDEX_PARSE_PATTERN));

        long start = startDate.toEpochDay();
        long end = endDate.toEpochDay();

        List<String> indexList = new ArrayList<>();
        while (start <= end) {

            String day = startDate.format(DateTimeFormatter.ofPattern(CDR_INDEX_FORMAT_PATTERN));
            indexList.add(CDR_INDEX_PREFIX + day + esAlias);

            startDate = startDate.plus(1, ChronoUnit.DAYS);
            start = startDate.toEpochDay();
        }

        return indexList;
    }
```
## 比较jestClien和elasticsearch-rest-high-level-client
两者是我在执行es命令时用过的java客户端，分别在两个产品中使用，前者的es version为5.4，后者的es version 为6.0，此时jestclient还不支持es 6.0,故选择elasticsearch-rest-high-level-client.JestClient是第三方开发的，后者则是elastic自家的，这边想吐槽的是elastic自家的在取出聚合数据时，诶嘛，费时费力，绕来老去，提供的数据格式有限，需自己手动转换，是在不如JestClient实用；而JestClient去aggs中的数据时，可直接转为json，借用fastjson获取数据，非常容易，特别是有过fastjon类似的序列化框架经验的开发人员。

## 教训
公司的产品还属于研发阶段，正式用户不多，不然也不会拖这么长的时间才发现问题，这次的教训够沉重的了，直接往生产环境es大量删除数据，如履薄冰，胆战心惊，就怕误删数据，这样的操作还是第一次...毕竟，引以为戒。








