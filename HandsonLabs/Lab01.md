
# 예제 1. Creating a container in Azure Cosmos DB

이 실습에서는 다양한 파티션 키와 설정을 사용하여 여러 Azure Cosmos DB 컨테이너를 만듭니다. 

## 1. Visual Studio Code에서 Cosmos DB Lab 환경 폴더 오픈   

1. Visual Studio Code 실행 전 Lab01외에 다른 Lab은 최상위 폴더로 이동 시켜둡니다.  
Lab0x 폴더들의 경로는 C:\Users\\[*사용자명*]\Documents\CosmosLabs\src\main\java\com\azure\cosmos\handsonlabs 아래에 있습니다.   
    <img src="https://user-images.githubusercontent.com/44718680/182098597-58150a8d-b017-4b45-94b4-ac54a3e22aa8.png"  width="600" height="400"/>   
    <img src="https://user-images.githubusercontent.com/44718680/182098884-b80b96b3-fc10-4e8c-a4dd-519420fe7e4e.png"  width="600" height="250"/>   

2. Visual Studio Code 실행   

3. your\home\directory\Documents\CosmosLabs 경로 폴더 오픈   
    <img src="https://user-images.githubusercontent.com/44718680/182083771-7fdd3600-882f-4ca9-945e-a79629791b31.png"  width="400" height="600"/>

4. 로딩된 프로젝트 파일 확인   
    <img src="https://user-images.githubusercontent.com/44718680/182101365-03086fbf-d394-4489-bdbd-fa4c401fbfb7.png"  width="620" height="450"/>   

5. pom.xml 파일에서 azure-cosmos의 dependency의 버전을 *LATEST* 로 변경   

    변경전   
     <img src="https://user-images.githubusercontent.com/44718680/182099756-ccc4c95b-9c45-4039-9533-b8c35b459a15.png"  width="800" height="350"/>        
    변경후   
     <img src="https://user-images.githubusercontent.com/44718680/182100655-4d913de1-c949-4c50-a60b-321c5db2c2ac.png"  width="800" height="350"/>   

6. handsonlabs\lab01\ 폴더의 Lab01Main.java 파일 오픈
    ![image](https://user-images.githubusercontent.com/44718680/182088478-72367395-6d0f-4953-a1c5-5562ad1e6dce.png)   
<br></br>

## 2. Lab01Main.java 편집 및 CosmosAsyncClient 인스턴스 생성 
CosmosAsyncClient 클래스는 Azure Cosmos DB에서 SQL API를 사용하기 위한 기본 "진입점"입니다. 
1. Lab01Main.java 편집기 탭 내에서 파일 상단에 다음 코드를 추가합니다.
```java
import com.azure.cosmos.handsonlabs.common.datatypes.ViewMap;
import com.azure.cosmos.models.*;

import com.azure.cosmos.ConsistencyLevel;
import com.azure.cosmos.CosmosAsyncClient;
import com.azure.cosmos.CosmosAsyncDatabase;
import com.azure.cosmos.CosmosAsyncContainer;
import com.azure.cosmos.CosmosClientBuilder;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.atomic.AtomicBoolean;
```
2. Lab01Main.java 클래스 내에서 다음 코드 줄을 추가하여 연결 정보에 대한 변수를 만듭니다.
```java
private static String endpointUri = "<your uri>";
private static String primaryKey = "<your key>";
```
3. 앞서 만든 Cosmos DB의 접근 주소(URI)와 키값이 필요합니다.
endpointUri 변수의 경우 자리 값을 URI 값으로 바꾸고 primaryKey 변수의 경우 값을 Azure Cosmos DB 계정의 PRIMARY KEY 값으로 바꿉니다.    
아직 값이 없는 경우 아래 링크의 지침을 사용하여 이 값을 가져옵니다.   
[Cosmos DB 엔드포인트, Key 확인](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop/blob/main/HandsonLabs/PreLab_%ED%99%98%EA%B2%BD_%EA%B5%AC%EC%84%B1.md#5-azure-%EB%B0%B0%ED%8F%AC-%EA%B2%B0%EA%B3%BC-%ED%99%95%EC%9D%B8)   
    > URI 샘플  
    >> private static String endpointUri = "https://cosmosacct.documents.azure.com:443/";   

    > 키 샘플   
    >> private static String primaryKey = "elzirrKCnXlacvh1CRAnQdYVbVLspmYHQyYrhx0PltHi8wn5lHVHFnd1Xm3ad5cn4TUcH4U0MSeHsVykkFPHpQ==";

4. main 메소드를 확인합니다.
```java
 public static void main(String[] args) {
 }
```
5. main 메서드 내에서 다음 코드 줄을 추가하여 CosmosAsyncClient 인스턴스를 만들도록 합니다.
```java
 CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .contentResponseOnWriteEnabled(true)
             .buildAsyncClient();
```
6. main 메소드에 클라이언트 종료 코드를 추가 합니다.
```java
 client.close();
```
7. 작성 완료된 Lab01Main 클래스를 확인 합니다.
```java
 public class Lab01Main {
     protected static Logger logger = LoggerFactory.getLogger(Lab01Main.class.getSimpleName());
     private static String endpointUri = "<your uri>";
     private static String primaryKey = "<your key>";    
     public static void main(String[] args) {
            
          CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .contentResponseOnWriteEnabled(true)
             .buildAsyncClient();

         client.close();        
     }
 }
 ```

8. 새 터미널 실행   
    <img src="https://user-images.githubusercontent.com/44718680/182104040-79a3ed8d-babb-4475-8870-93bf0fc9b0ff.png"  width="500" height="380"/>


9. clean package로 정상 빌드 여부를 확인 합니다. 
```mvn 
mvn clean package
```

## 3. SDK를 통한 데이터베이스와 컨테이너 생성 
컨테이너를 생성하려면 이름과 파티션 키 경로를 지정해야 합니다. 이 작업에서 컨테이너를 생성할 때 해당 값을 지정합니다. 
파티션 키는 분산된 기본 물리적 파티션 집합에 데이터를 배포하고 쿼리를 적절한 기본 파티션으로 효율적으로 라우팅하기 위한 논리적 힌트입니다. 

1. Lab01Main 클래스 정의의 맨 위에 데이터베이스 및 컨테이너 인스턴스에 대한 두 개의 정적 클래스 변수를 더 추가합니다.
```java
private static CosmosAsyncDatabase targetDatabase;
private static CosmosAsyncContainer customContainer;
private static AtomicBoolean resourcesCreated = new AtomicBoolean(false);
```

2. 아래의 코드를 main 메소드의 CosmosAsyncClient 생성과 client.close(); 코드 사이에 추가 합니다.
```java
client.createDatabaseIfNotExists("EntertainmentDatabase").flatMap(databaseResponse -> {
    targetDatabase = client.getDatabase(databaseResponse.getProperties().getId());
    CosmosContainerProperties containerProperties = 
        new CosmosContainerProperties("CustomCollection", "/type");
    return targetDatabase.createContainerIfNotExists(containerProperties, ThroughputProperties.createManualThroughput(1000));
}).flatMap(containerResponse -> {
    customContainer = targetDatabase.getContainer(containerResponse.getProperties().getId());
    return Mono.empty();
}).subscribe(voidItem -> {}, err -> {}, () -> {
    resourcesCreated.set(true);
});

while (!resourcesCreated.get());
```

3. while 반복문 아래에 아래 코드를 추가 합니다.   
```java
logger.info("Database Id:\t{}",targetDatabase.getId());
logger.info("Container Id:\t{}",customContainer.getId()); 
```

4. Lab01Main.java 변경 사항 저장  파일 우클릭 후 "Run Java" 실행   
    <img src="https://user-images.githubusercontent.com/44718680/182108533-31bbd764-c71f-4639-b747-aa53b5679b64.png"  width="400" height="600"/>
    
5. 실행 결과를 확인 합니다.   
    - Azure Portal에서 Cosmos DB 리소스에 데이터베이스 및 컨테이너 생성 여부 확인   

## 4. 컨테이너 사용자 지정 설정 적용 
1. 우리는 작성한 코드 중에서 아래 코드를 통해 컨테이너의 이름이 CustomCollection 이고 파티션키가 type 값이며 RU가 1,000으로 설정된 것을 확인 할 수 있습니다.   
**참고로 파티션 키는 대소문자를 구분하므로 주의해야 합니다.**
```java
 CosmosContainerProperties containerProperties = 
     new CosmosContainerProperties("CustomCollection", "/type");
 return targetDatabase.createContainerIfNotExists(containerProperties, ThroughputProperties.createManualThroughput(1000));
```
2. 이제 컨테이너의 설정을 변경해보기 위해 코드에서 아래와 같이 RU값을 2,000으로 변경해 봅니다.
```java
 CosmosContainerProperties containerProperties = 
     new CosmosContainerProperties("CustomCollection", "/type");
 return targetDatabase.createContainerIfNotExists(containerProperties, ThroughputProperties.createManualThroughput(2000));
``` 
3. 또한 인덱싱 정책을 설정하는 코드를 위 컨테이너 설정 코드 위에 추가 합니다.
```java
 IndexingPolicy indexingPolicy = new IndexingPolicy();
 indexingPolicy.setIndexingMode(IndexingMode.CONSISTENT);
 indexingPolicy.setAutomatic(true);
 List<IncludedPath> includedPaths = new ArrayList<>();
 IncludedPath includedPath = new IncludedPath("/*");
 includedPaths.add(includedPath);
 indexingPolicy.setIncludedPaths(includedPaths);   
```
4. 인덱싱 정책을 실제 적용하기 위해 CosmosContainerProperties 부분을 수정합니다. 
```java
 CosmosContainerProperties containerProperties = 
     new CosmosContainerProperties("CustomCollection", "/type");
 containerProperties.setIndexingPolicy(indexingPolicy);
 return targetDatabase.createContainerIfNotExists(containerProperties, ThroughputProperties.createManualThroughput(2000));    
```   

5. 완성 코드
```java
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

package com.azure.cosmos.handsonlabs.lab01;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import com.github.javafaker.Faker;
import java.math.BigDecimal;
import java.text.DecimalFormat;
import com.azure.cosmos.handsonlabs.common.datatypes.ViewMap;
import com.azure.cosmos.models.*;

import com.azure.cosmos.ConsistencyLevel;
import com.azure.cosmos.CosmosAsyncClient;
import com.azure.cosmos.CosmosAsyncDatabase;
import com.azure.cosmos.CosmosAsyncContainer;
import com.azure.cosmos.CosmosClientBuilder;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.atomic.AtomicBoolean;


public class Lab01Main {
    private static CosmosAsyncDatabase targetDatabase;
    private static CosmosAsyncContainer customContainer;
    private static AtomicBoolean resourcesCreated = new AtomicBoolean(false);
    protected static Logger logger = LoggerFactory.getLogger(Lab01Main.class.getSimpleName());
    private static String endpointUri = "";
    private static String primaryKey = "";
    public static void main(String[] args) {
        CosmosAsyncClient client = new CosmosClientBuilder()
        .endpoint(endpointUri)
        .key(primaryKey)
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .contentResponseOnWriteEnabled(true)
        .buildAsyncClient();

        IndexingPolicy indexingPolicy = new IndexingPolicy();
        indexingPolicy.setIndexingMode(IndexingMode.CONSISTENT);
        indexingPolicy.setAutomatic(true);
        List<IncludedPath> includedPaths = new ArrayList<>();
        IncludedPath includedPath = new IncludedPath("/*");
        includedPaths.add(includedPath);
        indexingPolicy.setIncludedPaths(includedPaths);  
       
        client.createDatabaseIfNotExists("EntertainmentDatabase").flatMap(databaseResponse -> {
            targetDatabase = client.getDatabase(databaseResponse.getProperties().getId());
            CosmosContainerProperties containerProperties = 
            new CosmosContainerProperties("CustomCollection", "/type");
        containerProperties.setIndexingPolicy(indexingPolicy);
        return targetDatabase.createContainerIfNotExists(containerProperties, ThroughputProperties.createManualThroughput(2000));    
        }).flatMap(containerResponse -> {
            customContainer = targetDatabase.getContainer(containerResponse.getProperties().getId());
            return Mono.empty();
        }).subscribe(voidItem -> {}, err -> {}, () -> {
            resourcesCreated.set(true);
        });
        
        while (!resourcesCreated.get());
        
        logger.info("Database Id:\t{}",targetDatabase.getId());
        logger.info("Container Id:\t{}",customContainer.getId()); 

    client.close();      
    }
}
```

5. 변경된 부분을 확인하기 위해 Azure portal에서 Cosmos DB리소스의 EntertainmentDatabase 데이터베이스를 삭제 후 해당 코드를 다시 수행해 봅니다. (Run Java)
<br></br>
<br></br>




# 예제 2. Populate a Container with Items using the SDK
## 1. 데이터 생성 
테스트 데이터 생성 코드를 작성해 보겠습니다.
1. Lab01Main.java 파일을 열어서 아래와 같이 CosmosAsyncClient 인스턴스 생성 코드와 client.close(); 사이의 코드를 모두 삭제 합니다.
```java
CosmosAsyncClient client = new CosmosClientBuilder()
           .endpoint(endpointUri)
           .key(primaryKey)
           .consistencyLevel(ConsistencyLevel.EVENTUAL)
           .contentResponseOnWriteEnabled(true)
           .buildAsyncClient();     
client.close();  
```
 
2. 필요한 라이브러리 사용을 위해 아래 코드를 추가 합니다.
```java
import com.azure.cosmos.handsonlabs.common.datatypes.PurchaseFoodOrBeverage;

import java.math.BigDecimal;
import java.text.DecimalFormat;
```

3. 아래 코드를 CosmosAsyncClient 인스턴스 생성 코드와 client.close(); 사이에 추가 합니다.
```java
targetDatabase = client.getDatabase("EntertainmentDatabase");
customContainer = targetDatabase.getContainer("CustomCollection");
```

4. 테스트 데이터 생성 코드를 추가 합니다.
```java
           ArrayList<PurchaseFoodOrBeverage> foodInteractions = new ArrayList<PurchaseFoodOrBeverage>();
            Faker faker = new Faker();    
               
            for (int i= 0; i < 500;i++){  
                PurchaseFoodOrBeverage doc = new PurchaseFoodOrBeverage(); 
                DecimalFormat df = new DecimalFormat("###.###");      
                doc.setType("PurchaseFoodOrBeverage");            
                doc.setQuantity(faker.random().nextInt(1, 5));            
                String unitPrice = df.format(Double.valueOf((Double)faker.random().nextDouble()));
                doc.setUnitPrice(new BigDecimal(unitPrice));
                int quantity = Integer.valueOf((Integer)doc.getQuantity());        
                String totalPrice = df.format(Double.valueOf(unitPrice) * quantity);
                doc.setTotalPrice(new BigDecimal(totalPrice));
                doc.setId(UUID.randomUUID().toString());
                foodInteractions.add(doc);
            }
            Flux<PurchaseFoodOrBeverage> foodInteractionsFlux = Flux.fromIterable(foodInteractions);
            List<CosmosItemResponse<PurchaseFoodOrBeverage>> results = 
                        foodInteractionsFlux.flatMap(interaction -> customContainer.createItem(interaction)).collectList().block();
                        
            results.forEach(result -> logger.info("Item Created\t{}",result.getItem().getId()));
```

5. 완성 코드
```java
package com.azure.cosmos.handsonlabs.lab01;

import com.azure.cosmos.handsonlabs.common.datatypes.PurchaseFoodOrBeverage;
import com.azure.cosmos.handsonlabs.common.datatypes.ViewMap;
import com.azure.cosmos.models.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import com.github.javafaker.Faker;

import com.azure.cosmos.ConsistencyLevel;
import com.azure.cosmos.CosmosAsyncClient;
import com.azure.cosmos.CosmosAsyncDatabase;
import com.azure.cosmos.CosmosAsyncContainer;
import com.azure.cosmos.CosmosClientBuilder;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.math.BigDecimal;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.atomic.AtomicBoolean;


public class Lab01Main {
    protected static Logger logger = LoggerFactory.getLogger(Lab01Main.class.getSimpleName());
    private static String endpointUri = "";
    private static String primaryKey = "";   
    private static CosmosAsyncDatabase targetDatabase;
    private static CosmosAsyncContainer customContainer;
    private static AtomicBoolean resourcesCreated = new AtomicBoolean(false);

    public static void main(String[] args) {
        CosmosAsyncClient client = new CosmosClientBuilder()
            .endpoint(endpointUri)
            .key(primaryKey)
            .consistencyLevel(ConsistencyLevel.EVENTUAL)
            .contentResponseOnWriteEnabled(true)
            .buildAsyncClient();
            targetDatabase = client.getDatabase("EntertainmentDatabase");
            customContainer = targetDatabase.getContainer("CustomCollection");

            ArrayList<PurchaseFoodOrBeverage> foodInteractions = new ArrayList<PurchaseFoodOrBeverage>();
            Faker faker = new Faker();    
               
            for (int i= 0; i < 500;i++){  
                PurchaseFoodOrBeverage doc = new PurchaseFoodOrBeverage(); 
                DecimalFormat df = new DecimalFormat("###.###");      
                doc.setType("PurchaseFoodOrBeverage");            
                doc.setQuantity(faker.random().nextInt(1, 5));            
                String unitPrice = df.format(Double.valueOf((Double)faker.random().nextDouble()));
                doc.setUnitPrice(new BigDecimal(unitPrice));
                int quantity = Integer.valueOf((Integer)doc.getQuantity());        
                String totalPrice = df.format(Double.valueOf(unitPrice) * quantity);
                doc.setTotalPrice(new BigDecimal(totalPrice));
                doc.setId(UUID.randomUUID().toString());
                foodInteractions.add(doc);
            }
            Flux<PurchaseFoodOrBeverage> foodInteractionsFlux = Flux.fromIterable(foodInteractions);
            List<CosmosItemResponse<PurchaseFoodOrBeverage>> results = 
                        foodInteractionsFlux.flatMap(interaction -> customContainer.createItem(interaction)).collectList().block();
                        
            results.forEach(result -> logger.info("Item Created\t{}",result.getItem().getId()));
            client.close();
    }
}
```

6. Lab01Main.java를 실행하여 결과를 확인 합니다.
- Azure portal에서 데이터 건수를 확인 합니다. 


## 2. 여러가지 유형의 데이터 생성 
1. 필요한 라이브러리 사용을 위해 아래 코드를 추가 합니다.
```java
import com.azure.cosmos.implementation.ConnectionPolicy;
import com.azure.cosmos.handsonlabs.common.datatypes.WatchLiveTelevisionChannel;
```
2. main 메소의 내용을 아래와 같이 수정합니다.   
<!---
Before   
```java
 public static void main(String[] args) {
    
     CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .buildAsyncClient();

     targetDatabase = client.getDatabase("EntertainmentDatabase");
     customContainer = targetDatabase.getContainer("CustomCollection");

     client.close();        
 }
```   

After   -->
```java
 public static void main(String[] args) {
     ConnectionPolicy defaultPolicy = ConnectionPolicy.getDefaultPolicy();
    
     CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .contentResponseOnWriteEnabled(true)
             .buildAsyncClient();

     targetDatabase = client.getDatabase("EntertainmentDatabase");
     customContainer = targetDatabase.getContainer("CustomCollection");

     ArrayList<WatchLiveTelevisionChannel> tvInteractions = new ArrayList<WatchLiveTelevisionChannel>();
     Faker faker = new Faker();

     for (int i= 0; i < 500;i++){  
         WatchLiveTelevisionChannel doc = new WatchLiveTelevisionChannel(); 

         doc.setChannelName(faker.funnyName().toString());
         doc.setMinutesViewed(faker.random().nextInt(1, 60));
         doc.setType("WatchLiveTelevisionChannel");
         doc.setId(UUID.randomUUID().toString());
         tvInteractions.add(doc);
     }

     Flux<WatchLiveTelevisionChannel> tvInteractionsFlux = Flux.fromIterable(tvInteractions);
     List<CosmosItemResponse<WatchLiveTelevisionChannel>> results = 
         tvInteractionsFlux.flatMap(interaction -> customContainer.createItem(interaction)).collectList().block();

     results.forEach(result -> logger.info("Item Created\t{}",result.getItem().getId()));

     client.close();        
 }
```   
3. 결과를 확인 합니다.   
- Azure portal에서 추가 입력된 데이터의 구조를 확인 합니다.   

Next Lab : [Importing Data into Azure Cosmos DB with Azure Data Factory](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop/blob/main/HandsonLabs/Lab02.md#importing-data-into-azure-cosmos-db-with-azure-data-factory)   
[목차로 돌아가기](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop#8-java-lab-guides)
