
# Creating a container in Azure Cosmos DB

이 실습에서는 다양한 파티션 키와 설정을 사용하여 여러 Azure Cosmos DB 컨테이너를 만듭니다. 

## 1. Visual Studio Code에서 Cosmos DB Lab 환경 폴더 오픈   

1. Visual Studio Code 실행 전 Lab01외에 다른 Lab은 최상위 폴더로 잠시 이동 시켜둡니다.   
    ![image](https://user-images.githubusercontent.com/44718680/182098597-58150a8d-b017-4b45-94b4-ac54a3e22aa8.png)   
    ![image](https://user-images.githubusercontent.com/44718680/182098884-b80b96b3-fc10-4e8c-a4dd-519420fe7e4e.png)   

2. Visual Studio Code 실행   

4. your\home\directory\Documents\CosmosLabs 경로 폴더 오픈   
    <img src="https://user-images.githubusercontent.com/44718680/182083771-7fdd3600-882f-4ca9-945e-a79629791b31.png"  width="400" height="600"/>

4. pom.xml 파일에서 azure-cosmos의 dependency의 버전을 *LATEST* 로 변경   

    변경전   
     ![image](https://user-images.githubusercontent.com/44718680/182099756-ccc4c95b-9c45-4039-9533-b8c35b459a15.png)   
    변경후   
     ![image](https://user-images.githubusercontent.com/44718680/182100655-4d913de1-c949-4c50-a60b-321c5db2c2ac.png)   


5. 로딩된 프로젝트 파일 확인   
    <img src="https://user-images.githubusercontent.com/44718680/182101365-03086fbf-d394-4489-bdbd-fa4c401fbfb7.png"  width="600" height="500"/>   

6. handsonlabs\lab01\ 폴더의 Lab01Main.java 파일 오픈
    ![image](https://user-images.githubusercontent.com/44718680/182088478-72367395-6d0f-4953-a1c5-5562ad1e6dce.png)   
<br></br>

## 2. CosmosAsyncClient 인스턴스 생성 
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
아직 값이 없는 경우 이 지침을 사용하여 이 값을 가져옵니다.
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


