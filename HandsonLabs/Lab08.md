# Intro to Azure Cosmos DB Change Feed

이 실습에서는 변경 피드(Change feed) 프로세서 라이브러리 및 Azure Functions를 사용하여 Azure Cosmos DB 변경 피드(Change feed)에 대한 세 가지 사용 사례를 구현합니다.

## 1. Build a Data Generator app to Generate Data
전자 상거래 웹 사이트의 작업 형태로 상점으로 흐르는 데이터를 시뮬레이션하기 위해 간단한 Java 클래스를 만들어 Cosmos DB CartContainer에 데이터를 추가합니다.

1. 앞서 다른 폴더로 이동 시켜두었던 Labs 폴더중에 lab08 폴더를 원래의 자리로 이동 시켜 줍니다.   
   C:\Users\\[*사용자명*]\Documents\CosmosLabs\src\main\java\com\azure\cosmos\handsonlabs 아래로 이동.   
   <img src="https://user-images.githubusercontent.com/44718680/182335649-203161a1-ca7b-44bc-a864-fd17158036dd.png"  width="630" height="380"/>
   <img src="https://user-images.githubusercontent.com/44718680/182335804-4194ddad-7ac6-4549-b6f8-45be2446b9a7.png"  width="630" height="180"/>

1. Visual Studio 를 엽니다.

2. your\home\directory\Documents\CosmosLabs 폴더를 오픈 합니다. 

3. 데이터 생성기 코드 작성을 위해 Lab8 폴더에서 New 파일 만들기로 DataGenerator.java 파일을 생성합니다.
   DataGenerator.java 파일에 아래 코드로 채웁니다.
   Lab08Main.java 파일은 Lab08Main으로 확장자를 잠시 없애 줍니다. 
   <img src="https://user-images.githubusercontent.com/44718680/182337708-6ac31074-6acf-419b-b7dc-d3762e6c3fd5.png"  width="300" height="450"/>

```java
package com.azure.cosmos.handsonlabs.lab08;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.Scanner;

import com.azure.cosmos.ConsistencyLevel;
import com.azure.cosmos.CosmosAsyncClient;
import com.azure.cosmos.CosmosAsyncContainer;
import com.azure.cosmos.CosmosAsyncDatabase;
import com.azure.cosmos.CosmosClientBuilder;
import com.azure.cosmos.handsonlabs.common.datatypes.ActionType;
import com.azure.cosmos.handsonlabs.common.datatypes.CartAction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import reactor.core.publisher.Flux;

public class DataGenerator {

    protected static Logger logger = LoggerFactory.getLogger(DataGenerator.class.getSimpleName());
    private static String endpointUri = "";
    private static String primaryKey = ""; 

    private static CosmosAsyncDatabase storeDatabase;
    private static CosmosAsyncContainer cartContainer;

    private static Random random = new Random();

    private static String[] items = new String[] {
            "Unisex Socks", "Women's Earring", "Women's Necklace", "Unisex Beanie",
            "Men's Baseball Hat", "Unisex Gloves", "Women's Flip Flop Shoes", "Women's Silver Necklace",
            "Men's Black Tee", "Men's Black Hoodie", "Women's Blue Sweater", "Women's Sweatpants",
            "Men's Athletic Shorts", "Women's Athletic Shorts", "Women's White Sweater", "Women's Green Sweater",
            "Men's Windbreaker Jacket", "Women's Sandal", "Women's Rainjacket", "Women's Denim Shorts",
            "Men's Fleece Jacket", "Women's Denim Jacket", "Men's Walking Shoes", "Women's Crewneck Sweater",
            "Men's Button-Up Shirt", "Women's Flannel Shirt", "Women's Light Jeans", "Men's Jeans",
            "Women's Dark Jeans", "Women's Red Top", "Men's White Shirt", "Women's Pant", "Women's Blazer Jacket",
            "Men's Puffy Jacket",
            "Women's Puffy Jacket", "Women's Athletic Shoes", "Men's Athletic Shoes", "Women's Black Dress",
            "Men's Suit Jacket", "Men's Suit Pant",
            "Women's High Heel Shoe", "Women's Cardigan Sweater", "Men's Dress Shoes", "Unisex Puffy Jacket",
            "Women's Red Dress", "Unisex Scarf",
            "Women's White Dress", "Unisex Sandals", "Women's Bag"
    };

    private static String[] states = new String[] {
            "AL", "AK", "AS", "AZ", "AR", "CA", "CO", "CT", "DE", "DC", "FM", "FL", "GA", "GU", "HI", "ID", "IL",
            "IN",
            "IA", "KS", "KY", "LA", "ME", "MH", "MD", "MA", "MI", "MN", "MS", "MO", "MT", "NE", "NV", "NH", "NJ",
            "NM",
            "NY", "NC", "ND", "MP", "OH", "OK", "OR", "PW", "PA", "PR", "RI", "SC", "SD", "TN", "TX", "UT", "VT",
            "VI",
            "VA", "WA", "WV", "WI", "WY"
    };

    private static double[] prices = new double[] {
            3.75, 8.00, 12.00, 10.00,
            17.00, 20.00, 14.00, 15.50,
            9.00, 25.00, 27.00, 21.00, 22.50,
            22.50, 32.00, 30.00, 49.99, 35.50,
            55.00, 50.00, 65.00, 31.99, 79.99,
            22.00, 19.99, 19.99, 80.00, 85.00,
            90.00, 33.00, 25.20, 40.00, 87.50, 99.99,
            95.99, 75.00, 70.00, 65.00, 92.00, 95.00,
            72.00, 25.00, 120.00, 105.00, 130.00, 29.99,
            84.99, 12.00, 37.50
    };

    public static void main(String[] args) {

        CosmosAsyncClient client = new CosmosClientBuilder()
                .endpoint(endpointUri)
                .key(primaryKey)
                .consistencyLevel(ConsistencyLevel.EVENTUAL)
                .contentResponseOnWriteEnabled(true)
                .buildAsyncClient();

        storeDatabase = client.getDatabase("StoreDatabase");
        cartContainer = storeDatabase.getContainer("CartContainer");

        logger.info("Enter number of documents to generate.");
        
        Scanner scanner = new Scanner(System.in);
        int noOfDocuments = scanner.nextInt();
        scanner.close();

        List<CartAction> cartActions = new ArrayList<CartAction>();
        for (int i = 0; i < noOfDocuments; i++) {
            cartActions.addAll(generateActions());
        }

        Flux<CartAction> cartActionFlux = Flux.fromIterable(cartActions);

        cartActionFlux.flatMap(item -> {
            return cartContainer.createItem(item);
        }).collectList().block();

        client.close();
    }

    private static List<CartAction> generateActions() {

        List<CartAction> actions = new ArrayList<CartAction>();

        int itemIndex = random.nextInt(items.length);
        int stateIndex = random.nextInt(states.length);
        int cartId = (random.nextInt(99999 - 1000) + 1000);
        ActionType actionType = ActionType.values()[random.nextInt(ActionType.values().length)];
        CartAction cartAction = new CartAction(cartId, actionType, items[itemIndex], prices[itemIndex],
                states[stateIndex]);

        if (cartAction.action != ActionType.Viewed) {
            List<ActionType> previousActions = new ArrayList<ActionType>();
            previousActions.add(ActionType.Viewed);

            if (cartAction.action == ActionType.Purchased) {
                previousActions.add(ActionType.Added);
            }

            previousActions.forEach(previousAction -> {
                CartAction previous = new CartAction(cartAction.cartId, previousAction, cartAction.item,
                        cartAction.price, cartAction.buyerState);
                actions.add(previous);
            });
        }

        actions.add(cartAction);
        return actions;
    }

}
```


4. Datagenerator.java에서 사용될 데이터 타입 추가를 위해 common\datatypes 폴더에 ActionType.java, CartAction.java 파일을 생성합니다. 
![image](https://user-images.githubusercontent.com/44718680/182517638-2e6b97b7-2792-49c8-b7ce-fef884d890cf.png)

5. ActionType.java 파일에 아래 코드로 채웁니다.
```java
package com.azure.cosmos.handsonlabs.common.datatypes;

public enum ActionType {
    Viewed,
    Added,
    Purchased
}
```

6. CartAction.java 파일에 아래 코드로 다. 
```java
package com.azure.cosmos.handsonlabs.common.datatypes;

import java.util.UUID;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties({"_rid", "_self", "_etag", "_attachments", "_lsn", "_ts"})
public class CartAction {

    public String id;
    public int cartId;
    public ActionType action;
    public String item;
    public double price;
    public String buyerState;

    public CartAction() {
        this.id = UUID.randomUUID().toString();
    }

    public CartAction(int cartId, ActionType actionType, String item, double price, String state) {
        this.id = UUID.randomUUID().toString();
        this.cartId = cartId;
        this.action = actionType;
        this.item = item;
        this.price = price;
        this.buyerState = state;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public int getCartId() {
        return cartId;
    }

    public void setCartId(int cartId) {
        this.cartId = cartId;
    }

    public ActionType getAction() {
        return action;
    }

    public void setAction(ActionType action) {
        this.action = action;
    }

    public String getItem() {
        return item;
    }

    public void setItem(String item) {
        this.item = item;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public String getBuyerState() {
        return buyerState;
    }

    public void setBuyerState(String buyerState) {
        this.buyerState = buyerState;
    }

}
```

7. DataGenerator.java 파일 우클릭 후 "Run Java"로 실행합니다.    
   실행 후 "INFO: Enter number of documents to generate." 메시지에서 입력하고자 하는 데이터 수 입력 후 엔터를 입력 합니다.
   
8. Cosmos DB 데이터 탐색기에서 입력된 데이터를 확인 합니다.  


## 2. Consume Cosmos DB Change Feed via the Change Feed Processor
Cosmos DB 변경 피드를 사용하기 위한 두 가지 주요 옵션은 Azure Functions 및 변경 피드 프로세서 라이브러리입니다. 
간단한 콘솔 애플리케이션을 통해 Change Feed Processor부터 테스트 하겠습니다.
본 테스트를 통해 Cosmos DB의 파티션키 변경을 위한 라이브 마이그레이션을 구현할 수 있습니다. 

### 1. Connect to the Cosmos DB Change Feed
1. Visual Studio Code를 엽니다. 

3. 데이터 생성기 코드 작성을 위해 Lab8 폴더에서 New 파일 만들기로 ChangeFeedMain.java 파일을 생성합니다.

4. ChangeFeedMain.java 파일을 아래 코드로 채웁니다.
```java
package com.azure.cosmos.handsonlabs.lab08;

import com.azure.cosmos.ChangeFeedProcessor;
import com.azure.cosmos.ChangeFeedProcessorBuilder;
import com.azure.cosmos.ConsistencyLevel;
import com.azure.cosmos.CosmosAsyncClient;
import com.azure.cosmos.CosmosAsyncContainer;
import com.azure.cosmos.CosmosAsyncDatabase;
import com.azure.cosmos.CosmosClientBuilder;
import com.azure.cosmos.models.ThroughputProperties;

import java.util.Scanner;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public class ChangeFeedMain {

    protected static Logger logger = LoggerFactory.getLogger(ChangeFeedMain.class.getSimpleName());
    private static String endpointUri = "";
    private static String primaryKey = ""; 
    private static CosmosAsyncDatabase storeDatabase;
    private static CosmosAsyncContainer cartContainer;
    private static CosmosAsyncContainer destinationContainer;
    private static CosmosAsyncContainer leaseContainer;

    public static void main(String[] args) {
        CosmosAsyncClient client = new CosmosClientBuilder()
                .endpoint(endpointUri)
                .key(primaryKey)
                .consistencyLevel(ConsistencyLevel.EVENTUAL)
                .contentResponseOnWriteEnabled(true)
                .buildAsyncClient();

        storeDatabase = client.getDatabase("StoreDatabase");
        cartContainer = storeDatabase.getContainer("CartContainer");
        destinationContainer = storeDatabase.getContainer("CartContainerByState");
        storeDatabase
                .createContainerIfNotExists("consoleLeases", "/id", ThroughputProperties.createManualThroughput(400))
                .flatMap(containerResponse -> {
                    leaseContainer = storeDatabase.getContainer(containerResponse.getProperties().getId());
                    return Mono.empty();
                }).block();

        ChangeFeedProcessor processor = new ChangeFeedProcessorBuilder()
                .hostName("host_1")
                .feedContainer(cartContainer)
                .leaseContainer(leaseContainer)
                .handleChanges(
                        docs -> {
                            logger.info("Changes received: " + docs.size());
                            Flux.fromIterable(docs).flatMap(doc -> destinationContainer.createItem(doc))
                                    .flatMap(itemResponse -> Mono.empty()).subscribe();
                        })
                .buildChangeFeedProcessor();

        processor.start().subscribe();

        logger.info("Started Change Feed Processor");
        logger.info("Press any key to stop the processor...");

        Scanner input = new Scanner(System.in);
        input.next();
        input.close();

        logger.info("Stopping Change Feed Processor");

        processor.stop().subscribe();
    }
}
```

5. ChangeFeedMain.java 파일 우클릭 후 "Run Java"로 실행합니다.    

6. DataGenerator.java를 실행하고 ChangeFeedMain.java 터미널 창의 로그 변화를 확인 합니다.

7. Cosmos DB 데이터 탐색기에서 입력된 데이터를 확인 합니다.  
   정상적으로 동작할 경우 DataGenerator.java에서 발생시킨 데이터가 StoreDatabase 데이터베이스의 CartContainer 컨테이너로 입력되며 ChangeFeedMain.java에 의해 CartContainerByState 컨테이너에도 데이터가 입력되는 것을 확인할 수 있습니다.


## 3. Create an Azure Function to Consume Cosmos DB Change Feed

아래 2개가 설치 되어야 합니다.   
[1. The Azure Functions Core Tools version 4.x.](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local#v2)   
[2.The Azure CLI version 2.4 or later.](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)   




