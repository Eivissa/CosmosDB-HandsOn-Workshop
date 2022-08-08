# Troubleshooting Performance in Azure Cosmos DB
>이 실습에서는 Java SDK를 사용하여 Azure Cosmos DB 요청을 조정하여 애플리케이션의 성능과 비용을 최적화합니다.   

## 1. Open the CosmosLabs Maven Project Template

1. 앞서 다른 폴더로 이동 시켜두었던 Labs 폴더중에 lab09 폴더를 원래의 자리로 이동 시켜 줍니다.   
   <img src="https://user-images.githubusercontent.com/44718680/183322155-03cfbdb1-b1f3-4640-b135-15cb5b770f40.png"  width="630" height="410"/>   
   <img src="https://user-images.githubusercontent.com/44718680/183322322-a77147f2-89e3-4258-b62a-9852abde491a.png"  width="660" height="220"/>   
   
2. Visual Studio를 엽니다.

3. your\home\directory\Documents\CosmosLabs 폴더를 오픈 합니다. 

## 2. Examining Response Headers
Azure Cosmos DB는 요청 및 서버 측에서 발생한 작업에 대한 추가 메타데이터를 제공할 수 있는 다양한 응답 헤더를 반환합니다.   
Java SDK는 이러한 헤더 중 많은 부분을 ResourceResponse<> 클래스의 속성으로 제공합니다.   

### 1. Observe RU Charge for Large Item

1. Lab09Main.java 파일 오픈 후 Main 메소드 내용을 아래 코드로 변경합니다.   
```java
 CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .contentResponseOnWriteEnabled(true)
             .buildAsyncClient();

 database = client.getDatabase("FinancialDatabase");
 peopleContainer = database.getContainer("PeopleCollection");
 transactionContainer = database.getContainer("TransactionCollection");         

 client.close();

```   

2. 아래 import 구문을 수정합니다.

Before   
```java
import com.azure.cosmos.ConnectionPolicy;
```
After   
```java
import com.azure.cosmos.implementation.ConnectionPolicy;
```   

3. 아래 import 구문을 삭제합니다.   
```java
import com.azure.cosmos.models.CosmosAsyncItemResponse;
```   

4. Person 클래스를 추가합니다.   
```java
import com.azure.cosmos.handsonlabs.common.datatypes.Person;
```   

5. Person 클래스 사용을 위해 아래 코드를 추가합니다.
```java
Person person = new Person(); 
```

6. person 변수를 CosmosAsyncContainer 인스턴스에서 createItem 메소들 사용해 입력하는 아래 코드를 추가합니다.
```java
CosmosItemResponse<Person> response = peopleContainer.createItem(person).block();
```   

7. 데이터 생성시 소모된 RU를 출력하기 위해 ItemResponse<> 인스턴스의 RequestCharge 속성 값을 출력합니다.   
```java
logger.info("{} RUs", response.getRequestCharge());
```   

8. 완성된 main 메소드 코드는 아래와 같습니다.   
```java
 public static void main(String[] args) {
    
     CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .contentResponseOnWriteEnabled(true)
             .buildAsyncClient();

     database = client.getDatabase("FinancialDatabase");
     peopleContainer = database.getContainer("PeopleCollection");
     transactionContainer = database.getContainer("TransactionCollection");
        
     Person person = new Person();
     CosmosItemResponse<Person> response =
     peopleContainer.createItem(person).block();
        
     logger.info("First item insert: {} RUs", response.getRequestCharge());

     client.close();        
 }
```   

9. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
   소모된 RU를 기록해 둡니다.  (13.9 RUs)   

10. Cosmos DB 데이터 탐색기에서 아래 쿼리를 수행하여 결과를 확인 합니다.   
```sql
SELECT TOP 2 * FROM coll ORDER BY coll._ts DESC
```   

11. main 메소드의 아래 코드를 수정합니다.   
Before     
```java
 Person person = new Person(); 
 CosmosItemResponse<Person> response = peopleContainer.createItem(person).block();

 logger.info("First item insert: {} RUs", response.getRequestCharge());
```
After   
```java
 List<Person> children = new ArrayList<Person>();
 for (int i=0; i<4; i++) children.add(new Person());
 Member member = new Member(UUID.randomUUID().toString(),
                             new Person(), // accountHolder
                             new Family(new Person(), // spouse
                                         children)); // children

 CosmosItemResponse<Member> response = peopleContainer.createItem(member).block();

 logger.info("Second item insert: {} RUs", response.getRequestCharge());                                            
```

<!--12. Member 클래스를 추가합니다.   
```java
import com.azure.cosmos.handsonlabs.common.datatypes.Member;
import com.azure.cosmos.handsonlabs.common.datatypes.Family;
```
-->


11. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.    
    소모된 RU를 기록해 둡니다.  (48.57 RUs)   

12. Cosmos DB 데이터 탐색기에서 아래 쿼리를 수행하여 결과를 확인 합니다.   
```sql
 SELECT * FROM coll WHERE IS_DEFINED(coll.relatives)
```   

### 2. Tune Index Policy

1. Azure Cosmos DB 블레이드에서 왼쪽에 있는 데이터 탐색기 링크를 찾아 클릭합니다.   

2. 데이터 탐색기 섹션에서 FinancialDatabase 데이터베이스 노드를 확장하고 PeopleCollection 노드를 확장한 다음 배율 및 설정(Scale & Settings) 옵션을 선택합니다.   

3. 설정 섹션에서 인덱싱 정책 필드를 찾아 현재 기본 인덱싱 정책을 관찰합니다.   
```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/\"_etag\"/?"
        }
    ],
    "spatialIndexes": [
        {
            "path": "/*",
            "types": [
                "Point",
                "LineString",
                "Polygon",
                "MultiPolygon"
            ]
        }
    ]
}
```   

> 이 정책은 JSON 문서의 모든 경로를 색인화합니다. 이 정책은 숫자(최대 8) 및 문자열(최대 100) 경로 모두에 대해 최대 정밀도(-1)를 구현합니다. 이 정책은 공간 데이터도 색인화합니다.   

4. 인덱싱 정책을 인덱스에서 /relatives/* 경로를 제거하는 새 정책으로 교체합니다.   
```json
 {
     "indexingMode": "consistent",
     "automatic": true,
     "includedPaths": [
         {
             "path":"/*",
             "indexes":[
                 {
                     "kind": "Range",
                     "dataType": "String",
                     "precision": -1
                 },
                 {
                     "kind": "Range",
                     "dataType": "Number",
                     "precision": -1
                 }
             ]
         }
     ],
     "excludedPaths": [
         {
             "path":"/\"_etag\"/?"
         },
         {
             "path":"/relatives/*"
         }
     ]
 }
```   
> 이 새로운 정책은 인덱스에서 큰 JSON 문서의 Children 속성을 효과적으로 제거하는 인덱싱에서 /relatives/* 경로를 제외합니다.   

5. 저장 버튼을 클릭하여 새 인덱싱 정책을 적용합니다.

6. Cosmos DB 데이터 탐색기에서 아래 쿼리를 수행하여 결과를 확인 합니다.
   여전히 결과는 기존과 동일함을 알 수 있습니다. 
```sql
SELECT * FROM coll WHERE IS_DEFINED(coll.relatives)
```

7. 아래 쿼리를 추가로 실행해 봅니다. 
```sql
SELECT * FROM coll WHERE IS_DEFINED(coll.relatives) ORDER BY coll.relatives.Spouse.FirstName
```
> 쿼리는 실패할 것 입니다. 이유는 ORDER BY 구문에는 인덱싱된 속성만 사용할 수 있습니다.


8. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.    
   소모된 RU를 기록해 둡니다.  (16.38 RUs)    


## 3. Troubleshooting Requests
먼저 Java SDK를 사용하여 컨테이너에 할당된 용량을 초과하는 요청을 발행합니다.   
요청 단위 소비는 초당 속도로 평가됩니다. 프로비저닝된 요청 단위 비율을 초과하는 애플리케이션의 경우 비율이 프로비저닝된 처리량 수준 아래로 떨어질 때까지 요청 비율이 제한됩니다.   
요청에 속도가 제한되면 서버는 HTTP 상태 코드 429 RequestRateTooLargeException으로 요청을 선제적으로 종료하고 x-ms-retry-after-ms 헤더를 반환합니다. 헤더는 클라이언트가 요청을 재시도하기 전에 기다려야 하는 시간(밀리초)을 나타냅니다.   
예제 애플리케이션에서 요청의 속도 제한을 관찰할 것입니다.   

### 1. Reducing RU Throughput for a Container   

1. Azure Portal에서 Cosmos DB 리소스를 찾습니다.   

2. FinancialDatabase 데이터베이스의 TransactionCollection 컨테이너의 Scale & Settings 옵션에서 Throughput 값을 400으로 수정합니다.   
  
### 2. Observing Throttling (HTTP 429)   

1. Lab09Main.java 파일의 main 메소드 안의 코드를 다음과 같이 수정합니다.   
```java
 public static void main(String[] args) {
     CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .contentResponseOnWriteEnabled(true)
             .buildAsyncClient();

     database = client.getDatabase("FinancialDatabase");
     peopleContainer = database.getContainer("PeopleCollection");
     transactionContainer = database.getContainer("TransactionCollection");         

     client.close();
 }
```   


2. TransactionCollection 컨테이너에 테스트 데이터를 생성하는 코드를 작성할 차례입니다.     
   아래 Transaction 인스턴스 데이터 생성 코드를 main 메소드 안에 추가합니다.   
```java
 List<Transaction> transactions = new ArrayList<Transaction>();
 for (int i=0; i<100; i++) transactions.add(new Transaction());
```   

3. Transaction 인스턴스 반복문을 추가합니다.   
```java
for (Transaction transaction : transactions) {

}
```   

4. foreach 블록 내에서 다음 코드 줄을 추가하여 항목을 비동기식으로 만들고 생성 작업의 결과를 변수에 저장합니다.   
```java
CosmosItemResponse<Transaction> result = transactionContainer.createItem(transaction).block();
```   
> CosmosAsyncContainer 클래스의 createItem 메서드는 JSON으로 직렬화하고 지정된 컬렉션 내 항목으로 저장하려는 개체를 가져옵니다.   

5. foreach 블록 내에서 다음 코드 줄을 추가하여 새로 생성된 리소스의 id 속성 값을 콘솔에 씁니다.   
```java
logger.info("Item Created {}", result.getItem().getId());
```   

6. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.    

7. 이 코드는 Async API를 사용하여 Azure Cosmos DB 컨테이너에 문서를 삽입하지만 각 createItem 호출을 차단하여 동기 방식으로 API를 사용하고 있습니다.   
한 스레드의 동기화 구현은 컨테이너 프로비저닝 처리량을 포화시킬 수 없습니다.   
이제 이러한 createItem 호출을 비동기 반응 프로그래밍 방식으로 다시 작성하고 컨테이너에 프로비저닝된 전체 400RU/s를 포화시킬 때 어떤 일이 발생하는지 살펴보겠습니다.   
코드 편집기 탭으로 돌아가서 다음 코드 줄을 찾습니다.    
```java
 for (Transaction transaction : transactions) {
     CosmosItemResponse<Transaction> result = transactionContainer.createItem(transaction).block();
     logger.info("Item Created {}", result.getItem().getId());
 }
```   

위 코드 부분을 아래 코드로 변경합니다. 
```java
 Flux<Transaction> interactionsFlux = Flux.fromIterable(transactions);
 List<CosmosItemResponse<Transaction>> results = 
     interactionsFlux.flatMap(interaction -> {
         return transactionContainer.createItem(interaction);
 })
 .collectList()
 .block();

 results.forEach(result -> logger.info("Item Created\t{}",result.getItem().getId()));
```   
> 위 코드는 생성 작업을 가능한 한 병렬로 실행하려고 시도할 것입니다. 컨테이너는 400RU/s로 구성되어 있습니다.    
> 이 구현에서는 Reactor 팩토리 메서드 Flux.fromIterable을 사용하여 트랜잭션 목록에서 Reactive Flux interactionFlux를 생성합니다.
그런 다음 이전 요청이 완료될 때까지 기다리지 않고 연속적인 createItem 요청을 발행하는 Reactive Streams 파이프라인에서 interactionFlux를 게시자로 사용합니다. 따라서 이것은 비동기 구현입니다. 이 코드 블록 끝에 있는 for-each 루프는 요청 결과를 반복하고 각각에 대한 알림을 인쇄합니다. 이러한 요청은 거의 병렬로 실행되므로 Azure Cosmos DB 컨테이너에 할당된 처리량이 요청 볼륨을 처리하기에 충분하지 않기 때문에 문서 수를 늘리면 예외적인 시나리오가 빠르게 발생해야 합니다.


8. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
> 이 쿼리는 성공적으로 실행되어야 합니다.   
> 단지 100개의 항목을 만들고 있으며 여기서 처리량 문제가 발생하지 않을 가능성이 높습니다.

9. 생성하는 문서의 수를 100개에서 5000개로 늘립니다. 

10. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     

### 3. Increasing RU Throughput to Reduce Throttling   

1. Azure Portal에서 Cosmos DB 리소스를 찾습니다.   

2. FinancialDatabase 데이터베이스의 TransactionCollection 컨테이너의 Scale & Settings 옵션에서 Throughput 값을 10000으로 수정합니다.   

<!--3. 생성하는 문서의 수를 5000개에서 10000개로 늘립니다. -->

3. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     

4. 테스트 완료 후 Throughput 값을 400으로 수정합니다.

## 4. Tuning Queries and Reads   
이제 Java SDK에서 RequestOptions 클래스의 SQL 쿼리 및 속성을 조작하여 Azure Cosmos DB에 대한 요청을 조정합니다.   

### 1. Measuring RU Charge   

1. Lab09Main.java 파일의 main 메소드 안의 코드를 다음과 같이 수정합니다.   
```java
 public static void main(String[] args) {
     CosmosAsyncClient client = new CosmosClientBuilder()
             .endpoint(endpointUri)
             .key(primaryKey)
             .consistencyLevel(ConsistencyLevel.EVENTUAL)
             .contentResponseOnWriteEnabled(true)
             .buildAsyncClient();

     database = client.getDatabase("FinancialDatabase");
     peopleContainer = database.getContainer("PeopleCollection");
     transactionContainer = database.getContainer("TransactionCollection");         

     client.close();
 }
```    

2. 문자열 변수에 SQL 쿼리를 저장할 다음 코드 줄을 추가합니다.   
```sql
String sql = "SELECT TOP 1000 * FROM c WHERE c.processed = true ORDER BY c.amount DESC";
```   
> 이 쿼리는 파티션 간 ORDER BY를 수행하고 상위 1000개 항목만 반환합니다.   

3. 첫번째 페이지의 결과만을 반환하기 위해 아래 코드를 추가합니다.   
```java
 CosmosQueryRequestOptions options = new CosmosQueryRequestOptions();
 transactionContainer.queryItems(sql, options, Transaction.class)
         .byPage()
         .next() // Take only the first page
         .flatMap(page -> {
         logger.info("Request Charge: {} RUs",page.getRequestCharge());
         return Mono.empty();
 }).block();
```    

4. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
   출력된 Request Charge를 확인   

5. 위 쿼리 구문을 아래 쿼리로 변경합니다.   
```java
String sql = "SELECT * FROM c WHERE c.processed = true";
```   

6. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
   출력된 Request Charge를 확인하고 차이를 확인합니다.   


7. 위 쿼리 구문을 아래 쿼리로 변경합니다.   
```java
String sql = "SELECT * FROM c";
```   

8. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
   출력된 Request Charge를 확인하고 차이를 확인합니다.   


9. 위 쿼리 구문을 아래 쿼리로 변경합니다.   
```java
String sql = "SELECT c.id FROM c";
```   

10. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
   출력된 Request Charge를 확인하고 차이를 확인합니다.   


### 2. Managing SDK Query Options   

1. Lab09Main.java 파일의 main 메소드 안의 코드를 다음과 같이 수정합니다.   
```java
public static void main(String[] args) {
  CosmosAsyncClient client = new CosmosClientBuilder()
          .endpoint(endpointUri)
          .key(primaryKey)
          .consistencyLevel(ConsistencyLevel.EVENTUAL)
          .contentResponseOnWriteEnabled(true)
          .buildAsyncClient();

  database = client.getDatabase("FinancialDatabase");
  peopleContainer = database.getContainer("PeopleCollection");
  transactionContainer = database.getContainer("TransactionCollection");         

  client.close();
}
```    

2. 다음 코드 줄을 추가하여 쿼리 옵션을 구성하는 변수를 만듭니다.   
```java
int maxItemCount = 10;
int maxDegreeOfParallelism = 1;
int maxBufferedItemCount = 0;
```   
 
3. 다음 코드 줄을 추가하여 변수에서 쿼리에 대한 옵션을 구성합니다.   
```java
CosmosQueryRequestOptions options = new CosmosQueryRequestOptions();
options.setMaxBufferedItemCount(maxBufferedItemCount);
options.setMaxDegreeOfParallelism(maxDegreeOfParallelism);
```   
 
4. 다음 코드를 추가하여 콘솔 창에 변수 값 출력을 작성합니다.   
```java
logger.info("\n\n" +
          "MaxItemCount:\t{}\n" +
          "MaxDegreeOfParallelism:\t{}\n" +
          "MaxBufferedItemCount:\t{}" + 
          "\n\n",
          maxItemCount, maxDegreeOfParallelism, maxBufferedItemCount);
```   

5. 문자열 변수에 SQL 쿼리를 저장할 다음 코드 줄을 추가합니다.   
```java
String sql = "SELECT * FROM c WHERE c.processed = true ORDER BY c.amount DESC";
```      

6. 수행시간 측정을 위한 타이머 코드를 추가합니다.   
```java
import org.apache.commons.lang3.time.StopWatch;

StopWatch timer = StopWatch.createStarted();
```   

7. 다음 코드 줄을 추가하여 항목 쿼리 인스턴스를 만들고 결과 집합을 열거합니다.   
```java
transactionContainer.queryItems(sql, options, Transaction.class)
      .byPage(maxItemCount)
      .flatMap(page -> {
      //Don't do anything with the query page results
      return Mono.empty();
}).blockLast();
```   

8. 타이머를 종료하는 코드를 추가합니다.   
```java
 timer.stop();
```   

9. 측정된 시간을 출력하는 코드를 추가합니다.   
```java
logger.info("\n\nElapsed Time:\t{}s\n\n", ((double)timer.getTime(TimeUnit.MILLISECONDS))/1000.0);
```   

10. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
    소모되는 RU량과 최종 수행 시간을 확인합니다.   

11. 처리 속도가 개선되는지를 확인하기 위해 병렬 처리수를 5로 수정합니다.   
```java
int maxDegreeOfParallelism = 5;
```   

12. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
    소모되는 RU량과 최종 수행 시간을 확인합니다.   


13. 처리 속도가 개선되는지를 확인하기 위해 데이터 미리가져오기 처리수를 -1로 수정합니다.   
```java
int maxBufferedItemCount = -1;
```   

14. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
    소모되는 RU량과 최종 수행 시간을 확인합니다.   


15. 처리 속도가 개선되는지를 확인하기 위해 병렬 처리수를 -1로 수정합니다.   
```java
int maxDegreeOfParallelism = -1;
```   

16. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
    소모되는 RU량과 최종 수행 시간을 확인합니다.   
 
17. 처리 속도가 개선되는지를 확인하기 위해 반환되는 아이템 처리수를 1000로 수정합니다.   
```java
int maxItemCount = 1000;
```   

18. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
    소모되는 RU량과 최종 수행 시간을 확인합니다.   

19. 처리 속도가 개선되는지를 확인하기 위해 데이터 미리가져오기 처리수를 50000로 수정합니다.   
```java
int maxBufferedItemCount = 50000;
```   

20. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
    소모되는 RU량과 최종 수행 시간을 확인합니다.   

### 3. Reading and Querying Items

1. Cosmos DB 데이터 익스플로러에서 FinancialDatabase 데이터베이스의 PeopleCollection 컨테이너의 데이터를 확인합니다.

2. 임의의 데이터 1건에 대한 id 속성 값과 파티션 키 값을 따로 기입해 둡니다. 

3. Lab09Main.java 파일의 main 메소드 안의 코드를 다음과 같이 수정합니다.   
```java
public static void main(String[] args) {
  CosmosAsyncClient client = new CosmosClientBuilder()
          .endpoint(endpointUri)
          .key(primaryKey)
          .consistencyLevel(ConsistencyLevel.EVENTUAL)
          .contentResponseOnWriteEnabled(true)
          .buildAsyncClient();

  database = client.getDatabase("FinancialDatabase");
  peopleContainer = database.getContainer("PeopleCollection");
  transactionContainer = database.getContainer("TransactionCollection");         

  client.close();
}
```    

4. 문자열 변수에 SQL 쿼리를 저장하는 다음 코드 줄을 추가합니다(example.document를 앞서 기록해 둔 id 값으로 바꿉니다).   
```sql
String sql = "SELECT TOP 1 * FROM c WHERE c.id = 'example.document'";
```   
> 이 쿼리는 지정된 고유 ID와 일치하는 단일 항목을 찾습니다.

5. 다음 코드 줄을 추가하여 항목 쿼리 인스턴스를 만들고 결과의 첫 페이지를 가져옵니다.    
   또한 페이지의 RequestCharge 속성 값과 검색된 항목의 내용을 인쇄합니다.   
```java
CosmosQueryRequestOptions options = new CosmosQueryRequestOptions();

peopleContainer.queryItems(sql, options, Member.class)
      .byPage()
      .next()
      .flatMap(page -> {
      logger.info("\n\n" +
                  "{} RUs for\n" +
                  "{}" + 
                  "\n\n",
                  page.getRequestCharge(),
                  page.getElements().iterator().next());
      return Mono.empty();
}).block();
```   
6. Lab09Main.java파일을 우클릭하고 Run Java를 수행하여 결과를 확인 합니다.     
> RU의 양이 표시되어야 합니다.

7. 다음 코드를 추가하여 CosmosAsyncContainer 클래스의 readItem 메서드를 사용하여 이전 단계에서 성으로 설정된 고유 ID와 파티션 키를 사용하여 항목을 검색합니다.    
RequestCharge 속성 값을 출력하는 행을 추가합니다.    
```java
int expectedWritesPerSec = 200;
int expectedReadsPerSec = 800;
double readRequestCharge = 0.0;
peopleContainer.readItem("example.document", new PartitionKey("<LastName>"), Member.class)
.flatMap(response -> {
  readRequestCharge = response.getRequestCharge();
  logger.info("\n\n{} RUs\n\n",readRequestCharge);
  return Mono.empty();
}).block();
```   



