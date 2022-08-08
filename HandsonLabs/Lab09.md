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

# 1. Observe RU Charge for Large Item

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



