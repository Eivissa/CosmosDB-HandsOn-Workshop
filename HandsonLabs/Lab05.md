# Building a Java Console App on Azure Cosmos DB
## 1. Build A Simple Java Console App

1. 앞서 다른 폴더로 이동 시켜두었던 Labs 폴더중에 lab05 폴더를 원래의 자리로 이동 시켜 줍니다.   
   C:\Users\\[*사용자명*]\Documents\CosmosLabs\src\main\java\com\azure\cosmos\handsonlabs 아래로 이동.   
   <img src="https://user-images.githubusercontent.com/44718680/182311971-22ab44a9-ede6-45ff-a406-1a5b05f29f12.png"  width="600" height="330"/>
   <img src="https://user-images.githubusercontent.com/44718680/182312123-16ec369a-2626-416d-b0fd-797f7fb980c6.png"  width="600" height="180"/>

1. Visual Studio를 엽니다.

2. your\home\directory\Documents\CosmosLabs 폴더를 오픈 합니다. 

3. lab05 폴더안의 Lab05Main.java 파일을 엽니다. 

4. pom.xml 파일을 열고 아래 코드를 추가 합니다.   
```xml
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>31.1-jre</version>
        </dependency>
```
<img src="https://user-images.githubusercontent.com/44718680/182317369-c4e6f98a-565e-4709-a06c-10b7e8d52059.png"  width="600" height="330"/>

5. package, import 구문 코드를 아래 내용으로 변경합니다.
```java
package com.azure.cosmos.handsonlabs.lab05;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.azure.cosmos.implementation.ConnectionPolicy;
import com.azure.cosmos.ConsistencyLevel;
import com.azure.cosmos.CosmosAsyncClient;
import com.azure.cosmos.CosmosAsyncDatabase;
import com.azure.cosmos.CosmosAsyncContainer;
import com.azure.cosmos.CosmosClientBuilder;
import com.azure.cosmos.handsonlabs.common.datatypes.Food;
import com.azure.cosmos.handsonlabs.common.datatypes.Serving;
import com.azure.cosmos.models.CosmosQueryRequestOptions;
import com.azure.cosmos.models.PartitionKey;

import reactor.core.publisher.Mono;
import java.util.concurrent.atomic.AtomicInteger;
import com.google.common.collect.Lists;
```

6. endpointUri, primaryKey 값을 Cosmos DB 키 URI와 키값으로 수정 합니다. 

7. main 메소드의 코드를 아래 코드로 변경 합니다. 
```java
        ConnectionPolicy defaultPolicy = ConnectionPolicy.getDefaultPolicy();
        defaultPolicy.setPreferredRegions(Lists.newArrayList("koreacentral"));
    
        CosmosAsyncClient client = new CosmosClientBuilder()
        .endpoint(endpointUri)
        .key(primaryKey)
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .contentResponseOnWriteEnabled(true)
        .buildAsyncClient();

        database = client.getDatabase("ImportDatabase");
        container = database.getContainer("FoodCollection");            
    
        container.readItem("19130", new PartitionKey("Sweets"), Food.class)
            .flatMap(candyResponse -> {
            Food candy = candyResponse.getItem();
            logger.info("Read {}",candy.getDescription());
            return Mono.empty();
        }).block();

        client.close();    
```

8. Lab05Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다. 

9. Cosmos DB 데이터 탐색기에서 아래 쿼리를 수행하여 결과를 확인 합니다.
```sql
  SELECT c.description FROM c WHERE c.id = "19130"
```
```sql
  SELECT c.description FROM c WHERE c.foodGroup = "Sweets"
```
