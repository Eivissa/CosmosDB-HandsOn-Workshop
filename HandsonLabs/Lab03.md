# Querying in Azure Cosmos DB
Azure Cosmos DB SQL API 계정은 가장 친숙하고 널리 사용되는 SQL(구조적 쿼리 언어)을 지원합니다. 이 랩에서는 Azure Portal을 통해 직접 이러한 풍부한 쿼리 기능을 사용하는 방법을 살펴봅니다. 별도의 도구나 클라이언트 측 코드가 필요하지 않습니다.

## 1. Query Overview
SQL을 사용하여 JSON을 쿼리하면 Azure Cosmos DB에서 레거시 관계형 데이터베이스의 장점을 NoSQL 데이터베이스와 결합할 수 있습니다. 하위 쿼리 또는 집계 함수와 같은 다양한 쿼리 기능을 사용할 수 있지만 NoSQL 데이터베이스에서 데이터 모델링의 많은 이점을 계속 유지할 수 있습니다.

Azure Cosmos DB는 엄격한 JSON 항목만 지원합니다. 유형 시스템 및 표현식은 JSON 유형만 처리하도록 제한됩니다. 자세한 내용은 [JSON 사양](https://www.json.org/)을 참조하세요.

### Importing data
Azure Data Factory를 사용하여 Azure Blob Storage의 Nutrition.json 파일에 저장된 JSON 배열 데이터를 Cosmos DB로 가져옵니다.

1. Cosmos DB 리소스 그룹에 미리 생성된 Data Factory 리소스를 사용할 수 있습니다. 

2. Data Factory의 Open Azure Data Factory Studio 버튼을 선택합니다. 
![image](https://user-images.githubusercontent.com/44718680/182300300-aeaee986-7224-4610-9a84-ad5000645819.png)

3. 수집 버튼을 선택 합니다. 
![image](https://user-images.githubusercontent.com/44718680/182300407-ad97631b-1869-4dab-a3e5-e70713c01e19.png)

4. 기본 제공 복사 작업을 선택합니다.   
   작업 주기 또는 작업 일정에서 지금 한 번 실행을 선택 합니다.   
   이후 다음를 선택 합니다. 
![image](https://user-images.githubusercontent.com/44718680/182300514-c81cee4b-f9d3-4084-8756-94825d36ec9a.png)

5. 원본 유형에서 Azure Blob Storage를 선택하고 새 연결을 선택합니다. 
![image](https://user-images.githubusercontent.com/44718680/182300933-ff1cc8e4-7e03-4706-982d-c6b3160f9eb3.png)

6. 이름을 NutritionJson로 입력하고 인증 방식을 **SAS URI** 방식으로 선택 후 아래 값을 SAS URL 부분에 붙여 넣습니다.   
   https://cosmoslabsstorageaccount.blob.core.windows.net/nutrition-data?si=container-list-read-policy&spr=https&sv=2021-06-08&sr=c&sig=jGrmrokYikbgbuW9we2am%2BwAq%2BC%2BxfZcPYswOeSQpAU%3D   
![image](https://user-images.githubusercontent.com/44718680/182301456-72ede8f4-e9f4-4fc4-866a-5c6af08810a0.png)

7. 파일 또는 폴더 텍스트 박스안에 nutrition-data 를 입력 후 찾아보기를 선택하여 NutritionData.json 파일을 선택합니다.
![image](https://user-images.githubusercontent.com/44718680/182303256-a4638a3a-589e-4fba-a835-d1ca0537f95b.png)

8. 옵션의 이진 복사, 재귀적, 파티션 검색 사용을 모두 체크 해제 한 후 다음을 클릭 합니다.
![image](https://user-images.githubusercontent.com/44718680/182303404-7889e819-84c4-4bc0-aceb-565c76023d32.png)

9. 파일 포맷은 JSON 포맷을 선택 후 다음을 클릭 합니다.
![image](https://user-images.githubusercontent.com/44718680/182303637-5e61dbc0-39db-4795-b623-91e8bc2b0776.png)

10. 대상 데이터 저장소의 경우 새 연결 만들기를 선택하고 Azure Cosmos DB(SQL API)를 선택하여 Cosmos DB 대상 데이터 저장소를 추가합니다.
![image](https://user-images.githubusercontent.com/44718680/182303717-1b3e3256-80dd-498b-a5ed-a85f6e444eb3.png)

11. 연결된 서비스의 이름을 targetcosmosdb로 지정하고 Azure 구독 및 Cosmos DB 계정을 선택합니다. 
    이전에 만든 Cosmos DB NutritionDatabase 를 선택 합니다.
    만들기를 클릭합니다.
![image](https://user-images.githubusercontent.com/44718680/191155007-48caef1a-0b66-47c4-b96a-d4899089c6fe.png)

12. 새로 만든 targetcosmosdb 연결을 대상 저장소로 선택합니다.   
    드롭다운 메뉴에서 FoodCollection 컨테이너를 선택합니다.   
    다음을 클릭합니다.   
![image](https://user-images.githubusercontent.com/44718680/182305156-7814df77-16ff-4c7e-b4c5-294513611e03.png)
 
13. 작업 이름을 ImportNutritionData로 지정합니다.   
    설정을 변경할 필요가 없습니다. 
![image](https://user-images.githubusercontent.com/44718680/182305388-5daa39ad-1f61-4b4e-8555-3c748f089d83.png)

14. 다음을 클릭하여 배포를 시작합니다.   
    배포가 완료되면 모니터를 선택합니다.
![image](https://user-images.githubusercontent.com/44718680/182305476-1fe22342-53a0-4594-aed4-35ca904d8638.png)
 
 15. 몇 분 후 페이지를 새로 고치면 ImportNutrition 파이프라인의 상태가 Succeeded로 나열되어야 합니다. 
     가져오기 프로세스가 완료되면 ADF를 닫습니다. 이제 가져온 데이터의 유효성 검사를 진행합니다.

## 3. Validate Imported Data

1. 리소스 그룹을 선택합니다.   
![image](https://user-images.githubusercontent.com/44718680/182298967-92f8ceb3-773e-48dc-afbb-3c7033bdc07a.png)   

2. Cosmos DB가 생성되어있는 리소스 그룹을 선택 합니다.   
![image](https://user-images.githubusercontent.com/44718680/182299036-1d829d6b-ec41-4f63-9a6b-696cbc22cda6.png)   

3. Cosmos DB 리소스를 선택 합니다.   
![image](https://user-images.githubusercontent.com/44718680/182299128-cc7d8aa1-fe5f-4351-b052-4e3431d5a2c9.png)   

4. Azure Cosmos DB 블레이드에서 블레이드 왼쪽에 있는 데이터 탐색기 링크를 찾아 클릭합니다.   
   NutritionDatabase 데이터베이스 노드를 확장한 다음 FoodCollection 컨테이너 노드를 확장합니다.   
![image](https://user-images.githubusercontent.com/44718680/191155412-c4394f7f-b28f-492a-b442-cfe16ecc5c36.png)   

5. FoodCollection 노드 내에서 항목(Items) 링크를 클릭하여 컨테이너에 있는 데이터를 확인합니다.   
![image](https://user-images.githubusercontent.com/44718680/191155516-c447922b-587d-4df8-8a7e-fc36fe81e401.png)



## 2. Running your first query    
### Open Data Explorer   
1. Azure Cosmos DB 블레이드에서 블레이드 왼쪽에 있는 데이터 탐색기(Data Explorer) 링크를 찾아 클릭합니다.   
2. 데이터 탐색기 섹션에서 NutritionDatabase 데이터베이스 노드를 확장한 다음 FoodCollection 컨테이너 노드를 확장합니다.   
3. FoodCollection 노드 내에서 항목(Items) 링크를 클릭합니다.   
4. 컨테이너 내의 데이터를 확인합니다.    
5. 새 SQL 쿼리를 클릭합니다.   
![image](https://user-images.githubusercontent.com/44718680/191158132-1699a21e-2a8f-4b4e-895e-ad887835e189.png)   
6. 다음 SQL 쿼리를 붙여넣고 쿼리 실행을 선택합니다.   
```sql
SELECT *
FROM food
WHERE food.foodGroup = "Snacks" and food.id = "19015"
```   
![image](https://user-images.githubusercontent.com/44718680/191158186-1736bfbf-d293-42c7-b1c9-7327a700bef8.png)   

쿼리가 id가 "19015"이고 foodGroup이 "Snacks"인 단일 문서를 반환한 것을 볼 수 있습니다.   
이 항목의 구조는 이 섹션의 나머지 부분에서 작업할 FoodCollection 컨테이너 내의 항목을 나타내므로 살펴봐야 합니다.   


## 3. Dot and quoted property projection accessors   
항목의 ID만 반환하려면 새 SQL 쿼리를 클릭하여 아래 쿼리를 실행할 수 있습니다.   
```sql 
SELECT food.id
FROM food
WHERE food.foodGroup = "Snacks" and food.id = "19015"
```   
아래와 같은 방법으로도 쿼리할 수 있습니다.
```sql
SELECT food["id"]
FROM food
WHERE food["foodGroup"] = "Snacks" and food["id"] = "19015"
```   

## 4. WHERE clauses
WHERE 절을 살펴보겠습니다. WHERE 절에 산술, 비교 및 논리 연산자를 포함한 복잡한 스칼라 표현식을 추가할 수 있습니다.

1. 새 SQL 쿼리를 클릭하여 아래 쿼리를 실행합니다. 다음 SQL 쿼리를 붙여넣고 쿼리 실행을 클릭합니다.   
```sql
SELECT food.id,
food.description,
food.tags,
food.foodGroup,
food.version
FROM food
WHERE (food.manufacturerName = "The Coca-Cola Company" AND food.version > 0)
```
이 쿼리는 manufacturerName = "The Coca-Cola Company"가 있고 version이 0보다 큰 항목에 대한 id, description, tags, foodGroup, version을 반환합니다.

## 5. Advanced projection
Azure Cosmos DB는 결과 JSON에서 여러 형태의 변환을 지원합니다.   
가장 간단한 방법 중 하나는 결과를 반환할 때 AS 별칭 지정 키워드를 사용하여 JSON 요소의 별칭을 지정하는 것입니다.   
   
아래 쿼리를 실행하면 요소 이름이 변환된 것을 볼 수 있습니다.   
또한 프로젝션은 WHERE 절로 지정된 모든 항목에 대해 제공 배열의 첫 번째 요소에만 액세스합니다.   
```sql
SELECT food.description,
food.foodGroup,
food.servings[0].description AS servingDescription,
food.servings[0].weightInGrams AS servingWeight
FROM food
WHERE food.foodGroup = "Fruits and Fruit Juices"
AND food.servings[0].description = "cup"
```   

## 6. ORDER BY clause   
Azure Cosmos DB는 하나 이상의 속성을 기반으로 결과를 정렬하기 위해 ORDER BY 절 추가를 지원합니다.   

```sql
SELECT food.description, 
food.foodGroup, 
food.servings[0].description AS servingDescription,
food.servings[0].weightInGrams AS servingWeight
FROM food
WHERE food.foodGroup = "Fruits and Fruit Juices" AND food.servings[0].description = "cup"
ORDER BY food.servings[0].weightInGrams DESC
```   
이후 인덱싱 랩에서 또는 [문서](https://docs.microsoft.com//azure/cosmos-db/sql-query-order-by)를 읽으면 Order By 절에 필요한 인덱스를 구성하는 방법에 대해 자세히 알아볼 수 있습니다.   

## 7. Limiting query result size   
Azure Cosmos DB는 TOP 키워드를 지원합니다.   
TOP은 쿼리에서 반환되는 값의 수를 제한하는 데 사용할 수 있습니다. 아래 쿼리를 실행하여 상위 20개 결과를 확인하세요.   
```sql
SELECT TOP 20 food.id,
food.description,
food.tags,
food.foodGroup
FROM food
WHERE food.foodGroup = "Snacks"
```   
OFFSET LIMIT 절은 건너뛰고 쿼리에서 일부 값을 가져오는 선택적 절입니다.    
아래 예제는 10번째 데이터 부터 10개의 데이터를 가져오는 예제 입니다.   
```sql
SELECT food.id,
food.description,
food.tags,
food.foodGroup
FROM food
WHERE food.foodGroup = "Snacks"
ORDER BY food.id
OFFSET 10 LIMIT 10
```   
OFFSET LIMIT를 ORDER BY 절과 함께 사용하면 정렬된 결과에서 데이터를 가져옵니다.

## 8. More advanced filtering   
쿼리에 IN 및 BETWEEN 키워드를 사용해 봅니다.   
IN은 지정된 값이 주어진 목록의 요소와 일치하는지 확인하는 데 사용할 수 있으며 BETWEEN은 값 범위에 대해 쿼리를 실행하는 데 사용할 수 있습니다.    
```sql
SELECT food.id,
       food.description,
       food.tags,
       food.foodGroup,
       food.version
FROM food
WHERE food.foodGroup IN ("Poultry Products", "Sausages and Luncheon Meats")
    AND (food.id BETWEEN "05740" AND "07050")
```

## 9. More advanced projection
Azure Cosmos DB는 쿼리 내에서 JSON 프로젝션을 지원합니다.   
속성 이름이 수정된 새 JSON 개체를 쿼리해 보겠습니다.    
아래 쿼리를 실행하여 결과를 확인하세요.   
```sql
SELECT { 
"Company": food.manufacturerName,
"Brand": food.commonName,
"Serving Description": food.servings[0].description,
"Serving in Grams": food.servings[0].weightInGrams,
"Food Group": food.foodGroup 
} AS Food
FROM food
WHERE food.id = "21421"
```   

## 10. JOIN within your documents   
Azure Cosmos DB의 JOIN은 문서 내 자체 조인을 지원합니다.    
Azure Cosmos DB는 문서 또는 컨테이너 간의 JOIN을 지원하지 않습니다.    

이전 쿼리 예제에서 우리는 food.servings 배열의 첫 번째 결과를 반환했습니다.   
아래의 조인 구문을 사용하여 배열 내의 모든 항목에 대한 결과에 항목을 반환할 수 있습니다.   
```sql
SELECT
food.id as FoodID,
serving.description as ServingDescription
FROM food
JOIN serving IN food.servings
WHERE food.id = "03226"
```   
JOIN은 배열 내의 속성을 필터링해야 하는 경우에 유용합니다.    
문서 내 JOIN 뒤에 필터가 있는 아래 예제를 실행합니다.   
```sql
SELECT VALUE COUNT(1)
FROM c
JOIN t IN c.tags
JOIN s IN c.servings
WHERE t.name = 'infant formula' AND s.amount > 1
```


## 11. System functions   
Azure Cosmos DB는 일반적인 작업을 위한 여러 기본 제공 기능을 지원합니다.   
ABS, FLOOR 및 ROUND와 같은 수학 함수와 IS_ARRAY, IS_BOOL 및 IS_DEFINED와 같은 유형 검사 기능을 다룹니다.   
[지원되는 시스템 기능](https://docs.microsoft.com/ko-kr/azure/cosmos-db/sql-query-system-functions)에 대해 링크에서 자세히 알아보십시오.   

일부 시스템 기능의 사용 예를 보려면 아래 쿼리를 실행하십시오.   
```sql
SELECT food.id,
food.commonName,
food.foodGroup,
ROUND(nutrient.nutritionValue) AS amount,
nutrient.units
FROM food JOIN nutrient IN food.nutrients
WHERE IS_DEFINED(food.commonName)
AND nutrient.description = "Water"
AND food.foodGroup IN ("Sausages and Luncheon Meats", "Legumes and Legume Products")
AND food.id > "42178"
```

## 12. Correlated subqueries
많은 시나리오에서 하위 쿼리가 효과적일 수 있습니다.   
상관 하위 쿼리는 외부 쿼리의 값을 참조하는 쿼리입니다.    
여기에서 가장 유용한 몇 가지 예를 살펴보겠습니다. 하위 쿼리에 대해 자세히 알아볼 수 있습니다.   

하위 쿼리에는 다중 값 하위 쿼리(Multi-value subqueries)와 스칼라 하위 쿼리(scalar subqueries)의 두 가지 유형이 있습니다.   
다중 값 하위 쿼리는 문서 세트를 반환하고 항상 FROM 절 내에서 사용됩니다. 스칼라 하위 쿼리 표현식은 단일 값으로 평가되는 하위 쿼리입니다.   

### 12-1. Multi-value subqueries
하위 쿼리를 사용하여 JOIN 표현식을 최적화할 수 있습니다.   

자체 조인을 수행한 다음 이름, NutritionValue 및 금액에 대한 필터를 적용하는 다음 쿼리를 고려하십시오.   
다음 표현식과 결합하기 전에 하위 쿼리를 사용하여 결합된 배열 항목을 필터링할 수 있습니다.   
```sql
SELECT VALUE COUNT(1)
FROM c
JOIN t IN c.tags
JOIN n IN c.nutrients
JOIN s IN c.servings
WHERE t.name = 'infant formula' AND (n.nutritionValue > 0 
AND n.nutritionValue < 10) AND s.amount > 1
```

3개의 하위 쿼리를 사용하여 이 쿼리를 다시 작성하여 RU(요청 단위) 요금을 최적화하고 줄일 수 있습니다.   
다중 값 하위 쿼리는 항상 외부 쿼리의 FROM 절에 나타납니다.   
```sql
SELECT VALUE COUNT(1)
FROM c
JOIN (SELECT VALUE t FROM t IN c.tags WHERE t.name = 'infant formula')
JOIN (SELECT VALUE n FROM n IN c.nutrients WHERE n.nutritionValue > 0 AND n.nutritionValue < 10)
JOIN (SELECT VALUE s FROM s IN c.servings WHERE s.amount > 1)
```

### 12-2. Scalar subqueries   
스칼라 하위 쿼리의 사용 사례 중 하나는 ARRAY_CONTAINS를 EXISTS로 다시 작성하는 것입니다.   
ARRAY_CONTAINS를 사용하는 다음 쿼리를 고려하십시오.   
```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE ARRAY_CONTAINS(f.tags, {name: 'orange'})
```
결과는 같지만 EXISTS를 사용하는 다음 쿼리를 실행합니다.   
```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE EXISTS(SELECT VALUE t FROM t IN f.tags WHERE t.name = 'orange')
```   
EXISTS 사용의 주요 이점은 ARRAY_CONTAINS가 허용하는 단순한 동등 필터가 아니라 EXISTS 함수에 복잡한 필터를 가질 수 있다는 것입니다.   




Next Lab : [Indexing in Azure Cosmos DB](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop/blob/main/HandsonLabs/Lab04.md#indexing-in-azure-cosmos-db)   
[목차로 돌아가기](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop#8-java-lab-guides)











