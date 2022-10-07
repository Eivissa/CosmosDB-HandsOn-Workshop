# Building a Java Console App on Azure Cosmos DB
이전에는 Azure Portal의 Data Explorer를 사용하여 Azure Cosmos DB 컨테이너를 쿼리했습니다.    
이제 Java SDK를 사용하여 유사한 쿼리를 실행할 것입니다.   

### 1. Read a single Document in Azure Cosmos DB Using readItem()
1. 앞서 다른 폴더로 이동 시켜두었던 Labs 폴더중에 lab05 폴더를 원래의 자리로 이동 시켜 줍니다.   
   C:\Users\\[*사용자명*]\Documents\CosmosLabs\src\main\java\com\azure\cosmos\handsonlabs 아래로 이동.   
   <img src="https://user-images.githubusercontent.com/44718680/182311971-22ab44a9-ede6-45ff-a406-1a5b05f29f12.png"  width="600" height="330"/>
   <img src="https://user-images.githubusercontent.com/44718680/182312123-16ec369a-2626-416d-b0fd-797f7fb980c6.png"  width="600" height="180"/>

2. Visual Studio를 엽니다.

3. your\home\directory\Documents\CosmosLabs 폴더를 오픈 합니다. 

4. lab05 폴더안의 Lab05Main.java 파일을 엽니다. 

5. pom.xml 파일을 열고 아래 코드를 추가 합니다.   
```xml
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>31.1-jre</version>
        </dependency>
```
<img src="https://user-images.githubusercontent.com/44718680/182317369-c4e6f98a-565e-4709-a06c-10b7e8d52059.png"  width="600" height="330"/>

6. package, import 구문 코드를 아래 내용으로 변경합니다.
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

7. endpointUri, primaryKey 값을 Cosmos DB 키 URI와 키값으로 수정 합니다. 

8. main 메소드의 코드를 아래 코드로 변경 합니다. 
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

9. Lab05Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.   
readItem 함수를 통해 조회한 데이터가 출력됩니다.

10. Cosmos DB 데이터 탐색기에서 아래 쿼리를 수행하여 결과를 확인 합니다.
```sql
  SELECT c.description FROM c WHERE c.id = "19130"
```
```sql
  SELECT c.description FROM c WHERE c.foodGroup = "Sweets"
```
<BR></BR>
<BR></BR>
### 2. Execute a Query Against a Single Azure Cosmos DB Partition
1. 다음으로 코드를 추가 해야 할 부분을 확인합니다.  
```java
 container.readItem("19130", new PartitionKey("Sweets"), Food.class)
         .flatMap(candyResponse -> {
         Food candy = candyResponse.getItem();
         logger.info("Read {}",candy.getDescription());
         return Mono.empty();
 }).block();
    
 // <== New code goes here
```

2. 다음 SQL 쿼리 정의를 위에서 확인한 코드 추가 부분에 추가 합니다.
```java
   String sqlA = "SELECT f.description, f.manufacturerName, f.servings FROM foods f WHERE f.foodGroup = 'Sweets' and IS_DEFINED(f.description) and IS_DEFINED(f.manufacturerName) and IS_DEFINED(f.servings)";

```
이 쿼리는 foodGroup이 Sweets 값으로 설정된 모든 음식을 선택합니다. 
또한 description, manufacturerName 및 servings 속성이 정의된 문서만 선택합니다.
이 쿼리에는 WHERE 절에 파티션 키가 있기 때문에 이 쿼리는 단일 파티션 내에서 실행할 수 있습니다.
참고: https://docs.microsoft.com/ko-kr/azure/cosmos-db/sql/sql-query-is-defined 

3. 아래 코드를 쿼리 정의 밑에 추가합니다.
```java
 CosmosQueryRequestOptions optionsA = new CosmosQueryRequestOptions();
 optionsA.setMaxDegreeOfParallelism(1);
 container.queryItems(sqlA, optionsA, Food.class).byPage()
         .flatMap(page -> {
         for (Food fd : page.getResults()) {
             String msg="";
             msg = String.format("%s by %s\n",fd.getDescription(),fd.getManufacturerName());

             for (Serving sv : fd.getServings()) {
                 msg += String.format("\t%f %s\n",sv.getAmount(),sv.getDescription());
             }
             msg += "\n";
             logger.info(msg);
         }

         return Mono.empty();
 }).blockLast();
```

4. Lab05Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다. 

5. Cosmos DB 데이터 탐색기에서 아래 쿼리를 수행하여 결과를 확인 합니다.
```sql
   SELECT f.description, f.manufacturerName, f.servings FROM foods f WHERE f.foodGroup = 'Sweets' and IS_DEFINED(f.description) and IS_DEFINED(f.manufacturerName) and IS_DEFINED(f.servings)

```   
<BR></BR>
<BR></BR>
### 3. Execute a Query Against Multiple Azure Cosmos DB Partitions

1. 앞서 작성한 코드를 복사하여 중복되도록 붙여 넣습니다.
```java
        String sqlA = "SELECT f.description, f.manufacturerName, f.servings FROM foods f WHERE f.foodGroup = 'Sweets' and IS_DEFINED(f.description) and IS_DEFINED(f.manufacturerName) and IS_DEFINED(f.servings)";
        
        CosmosQueryRequestOptions optionsA = new CosmosQueryRequestOptions();
        optionsA.setMaxDegreeOfParallelism(1);
        container.queryItems(sqlA, optionsA, Food.class).byPage()
                .flatMap(page -> {
                for (Food fd : page.getResults()) {
                    String msg="";
                    msg = String.format("%s by %s\n",fd.getDescription(),fd.getManufacturerName());
       
                    for (Serving sv : fd.getServings()) {
                        msg += String.format("\t%f %s\n",sv.getAmount(),sv.getDescription());
                    }
                    msg += "\n";
                    logger.info(msg);
                }
       
                return Mono.empty();
        }).blockLast();

        String sqlA = "SELECT f.description, f.manufacturerName, f.servings FROM foods f WHERE f.foodGroup = 'Sweets' and IS_DEFINED(f.description) and IS_DEFINED(f.manufacturerName) and IS_DEFINED(f.servings)";
        
        CosmosQueryRequestOptions optionsA = new CosmosQueryRequestOptions();
        optionsA.setMaxDegreeOfParallelism(1);
        container.queryItems(sqlA, optionsA, Food.class).byPage()
                .flatMap(page -> {
                for (Food fd : page.getResults()) {
                    String msg="";
                    msg = String.format("%s by %s\n",fd.getDescription(),fd.getManufacturerName());
       
                    for (Serving sv : fd.getServings()) {
                        msg += String.format("\t%f %s\n",sv.getAmount(),sv.getDescription());
                    }
                    msg += "\n";
                    logger.info(msg);
                }
       
                return Mono.empty();
        }).blockLast();

```

2. 중복 쿼리 코드 내에서 문자열 sqlA를 정의한 위치를 찾아 아래와 같이 새 쿼리 문자열 sqlB로 바꿉니다.
```sql
String sqlB = "SELECT f.id, f.description, f.manufacturerName, f.servings FROM foods f WHERE IS_DEFINED(f.manufacturerName)";
```

3. 이 쿼리의 경우 더 큰 동시성을 사용하여 실행하고 최대 항목 수를 100 개로 허용하도록 수정합니다.    
   또한 코드의 optionsA, sqlA를 optionB, sqlB로 변경해야 합니다.

Before
```java
CosmosQueryRequestOptions optionsA = new CosmosQueryRequestOptions();
optionsA.setMaxDegreeOfParallelism(1);
container.queryItems(sqlA, optionsA, Food.class).byPage()
```   
After
```java
CosmosQueryRequestOptions optionsB = new CosmosQueryRequestOptions();
optionsB.setMaxDegreeOfParallelism(5);
container.queryItems(sqlB, optionsB, Food.class).byPage(100)
```

4. 아래 코드를 클래스 변수 선언부에 추가 합니다.
```java
 private static AtomicInteger pageCount = new AtomicInteger(0);
```
   thread-safe page count를 처리하기 위함입니다.

5. 아래 코드 부분을 찾습니다.
```java
 for (Food fd : page.getResults()) {
     String msg="";
     msg = String.format("%s by %s\n",fd.getDescription(),fd.getManufacturerName());

     for (Serving sv : fd.getServings()) {
         msg += String.format("\t%f %s\n",sv.getAmount(),sv.getDescription());
     }
     msg += "\n";
     logger.info(msg);
 } 
```
그리고 아래 코드로 바꾸어 줍니다.
```java
 String msg="";

 msg = String.format("---Page %d---\n",pageCount.getAndIncrement());

 for (Food fd : page.getResults()) {
     msg += String.format("\t[%s]\t%s\t%s\n",fd.getId(),fd.getDescription(),fd.getManufacturerName());
 }
 logger.info(msg);    
```

6. Lab05Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다. 

7. Cosmos DB 데이터 탐색기에서 쿼리를 수행하여 결과를 확인 합니다.


Next Lab : [Intro to Azure Cosmos DB Change Feed](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop/blob/main/HandsonLabs/Lab08.md#intro-to-azure-cosmos-db-change-feed)   
[목차로 돌아가기](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop#8-java-lab-guides)

