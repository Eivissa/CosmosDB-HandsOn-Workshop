# Indexing in Azure Cosmos DB   
이 실습에서는 Azure Cosmos DB 컨테이너의 인덱싱 정책을 수정합니다.    
쓰기 또는 읽기 작업량이 많은 워크로드에 대해 인덱싱 정책을 최적화하는 방법과 다양한 SQL API 쿼리 기능에 대한 인덱싱 요구 사항을 이해하는 방법을 살펴봅니다.   

## 1. Indexing Overview   
Azure Cosmos DB는 스키마 또는 인덱스 관리를 처리할 필요 없이 사용할 수 있는 스키마에 구애받지 않는 데이터베이스입니다.    
기본적으로 Azure Cosmos DB는 스키마를 정의하거나 보조 인덱스를 구성할 필요 없이 컨테이너의 모든 항목에 대한 모든 속성을 자동으로 인덱싱합니다.    
인덱싱 정책을 기본 설정으로 유지하도록 선택한 경우 최적의 성능으로 대부분의 쿼리를 실행할 수 있으며 인덱싱을 명시적으로 고려할 필요가 없습니다.    
인덱스에서 속성 추가 또는 제거를 제어하려는 경우 Azure Portal 또는 SQL API SDK를 통해 수정할 수 있습니다.   

Azure Cosmos DB는 데이터를 트리 형식으로 나타내는 Inverted 인덱스를 사용합니다.    
작동 방식에 대한 간략한 소개를 보려면 실습을 계속하기 전에 [인덱싱 개요](https://docs.microsoft.com/en-us/azure/cosmos-db/index-overview)를 읽으십시오.   

## 2. Customizing the indexing policy    
이 실습 섹션에서는 FoodCollection에 대한 인덱싱 정책을 보고 수정합니다.   

### 2.1. Open Data Explorer   
1. Azure Cosmos DB 블레이드에서 데이터 탐색기(Data Explorer) 링크를 찾아 클릭합니다.   
2. 데이터 탐색기 섹션에서 NutritionDatabase 데이터베이스 노드를 확장한 다음 FoodCollection 컨테이너 노드를 확장합니다.   
3. FoodCollection 노드 내에서 항목(Items)을 클릭합니다.   
4. 컨테이너 내의 항목을 봅니다. 이러한 문서에 배열을 포함하여 많은 속성이 있는 방법을 관찰하십시오. WHERE 절, ORDER BY 절 또는 JOIN에서 특정 속성을 사용하지 않는 경우 속성을 인덱싱해도 성능상의 이점이 없습니다.   
5. FoodCollection 노드 내에서 Scale & Settings 링크를 클릭합니다. 인덱싱 정책 섹션에서 컨테이너의 인덱스를 정의하는 JSON 파일을 편집할 수 있습니다. 인덱싱 정책은 모든 Azure Cosmos DB SDK를 통해 수정할 수도 있지만 이 실습에서는 Azure Portal을 통해 인덱싱 정책을 수정합니다.   

### 2.2. Including and excluding Range Indexes   
기본적으로 모든 속성에 범위 인덱스를 포함하는 대신 인덱스에서 특정 경로를 포함하거나 제외하도록 선택할 수 있습니다.   
몇 가지 간단한 예를 살펴보겠습니다.   

FoodCollection 내에서 문서에는 다음 스키마가 있습니다(가독성을 위해 일부 속성이 제거됨).   
```json
{
    "id": "36000",
    "_rid": "LYwNAKzLG9ADAAAAAAAAAA==",
    "_self": "dbs/LYwNAA==/colls/LYwNAKzLG9A=/docs/LYwNAKzLG9ADAAAAAAAAAA==/",
    "_etag": "\"0b008d85-0000-0700-0000-5d1a47e60000\"",
    "description": "APPLEBEE'S, 9 oz house sirloin steak",
    "tags": [
        {
            "name": "applebee's"
        },
        {
            "name": "9 oz house sirloin steak"
        }
    ],

    "manufacturerName": "Applebee's",
    "foodGroup": "Restaurant Foods",
    "nutrients": [
        {
            "id": "301",
            "description": "Calcium, Ca",
            "nutritionValue": 16,
            "units": "mg"
        },
        {
            "id": "312",
            "description": "Copper, Cu",
            "nutritionValue": 0.076,
            "units": "mg"
        },
    ]
} 
```   
범위 인덱스가 있는 manufacturerName, foodGroup, nutrients 속성만 인덱싱하려면 다음 인덱스 정책을 정의해야 합니다.   
```json
{
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/manufacturerName/*"
            },
            {
                "path": "/foodGroup/*"
            },
            {
                "path": "/nutrients/[]/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```
이 예에서 와일드카드 문자 '\*'를 사용하여 nutrients 배열 내의 모든 경로를 인덱싱하려는 것을 나타냅니다.    
그러나 각 배열 요소 중 NutritionValue를 색인화하고 싶을 수도 있습니다.   

다음 예에서 인덱싱 정책은 nutrients 배열의 NutritionValue 경로가 인덱싱되어야 함을 명시적으로 지정합니다.    
와일드카드 문자 '\*'를 사용하지 않기 때문에 배열의 추가 경로는 인덱싱되지 않습니다.   
```json
{
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/manufacturerName/*"
            },
            {
                "path": "/foodGroup/*"
            },
            {
                "path": "/nutrients/[]/nutritionValue/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

마지막으로 \*와 ?의 차이점을 이해하는 것이 중요합니다.    
\* 문자는 Azure Cosmos DB가 해당 특정 노드를 넘어서는 모든 경로를 인덱싱해야 함을 나타냅니다.    
? 문자는 Azure Cosmos DB가 이 노드를 넘어서는 더 이상 경로를 인덱싱하지 않아야 함을 나타냅니다. 
위의 예에서 NutritionValue 아래에는 추가 경로가 없습니다.   
문서를 수정하여 여기에 경로를 추가하는 경우 위의 예에서 와일드카드 문자 '\*'를 사용하면 인덱싱 정책에 이름을 명시적으로 언급하지 않고도 속성이 인덱싱됩니다.  


### 2.3. Understand query requirements   
인덱싱 정책을 수정하기 전에 데이터가 어떻게 사용되는지 이해하는 것이 중요합니다.   
워크로드가 쓰기 작업이 많거나 문서가 큰 경우 필요한 경로만 인덱싱해야 합니다. 이렇게 하면 삽입, 업데이트 및 삭제에 필요한 RU의 양이 크게 줄어듭니다.   

다음 쿼리가 FoodCollection 컨테이너에서 실행되는 유일한 읽기 작업이라고 가정해 보겠습니다.   

Query #1
```sql 
SELECT * FROM c WHERE c.manufacturerName = <manufacturerName>
```
Query #2
```sql
SELECT * FROM c WHERE c.foodGroup = <foodGroup>
```
이러한 쿼리는 manufacturerName 및 foodGroup에 각각 범위 인덱스를 정의하기만 하면 됩니다. 이러한 속성만 인덱싱하도록 인덱싱 정책을 수정할 수 있습니다.

### 2.4. Edit the indexing policy by including paths
Azure Portal에서 FoodCollection으로 다시 이동하고 Scale & Settings 링크를 클릭합니다. 
인덱싱 정책 섹션에서 내용을 다음으로 바꿉니다.
```json
{
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/manufacturerName/*"
            },
            {
                "path": "/foodGroup/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```
이 새로운 인덱싱 정책은 ManufacturerName 및 foodGroup 속성에 대해서만 범위 인덱스를 생성합니다. 
다른 모든 속성에서 범위 인덱스를 제거합니다. 
저장을 클릭합니다. 

컨테이너를 다시 인덱싱하는 동안 쓰기 성능은 영향을 받지 않습니다. 그러나 쿼리는 불완전한 결과를 반환할 수 있습니다.   

1. 새 인덱싱 정책을 정의한 후 FoodCollection으로 이동하여 새 SQL 쿼리 추가 아이콘을 선택합니다. 다음 SQL 쿼리를 붙여넣고 쿼리 실행을 선택합니다.   
```sql
SELECT * FROM c WHERE c.manufacturerName = "Kellogg, Co."
```

쿼리 통계 탭으로 이동합니다. 인덱스에서 일부 속성을 제거한 후에도 이 쿼리의 RU 요금이 낮다는 점을 관찰해야 합니다.   
manufacturerName은 쿼리에서 필터로 사용된 유일한 속성이었기 때문에 필요한 유일한 인덱스였습니다.   

이제 쿼리 텍스트를 다음으로 바꾸고 쿼리 실행을 선택합니다.
```sql
SELECT * FROM c WHERE c.description = "Bread, blue corn, somiviki (Hopi)"
```
단일 문서만 반환되는 경우에도 이 쿼리는 RU 요금이 매우 높다는 점을 관찰해야 합니다.   
이는 현재 description 속성에 대해 정의된 범위 인덱스가 없기 때문입니다.   

또한 아래의 쿼리 메트릭을 관찰하십시오.   
![image](https://user-images.githubusercontent.com/44718680/191183610-00baacf8-b55f-494d-bf93-570f09182e24.png)   
Retrieved document count가 8618 Index hit document count가 0 입니다.
인덱스를 사용할 수 없으므로 8618건의 데이터를 모두 검색했습니다.

### 2.4. Edit the indexing policy by excluding paths
인덱싱할 특정 경로를 수동으로 포함하는 것 외에도 특정 경로를 제외할 수 있습니다. 
대부분의 경우 이 접근 방식을 사용하면 기본적으로 문서의 모든 새 속성을 인덱싱할 수 있으므로 더 간단할 수 있습니다. 
쿼리에 절대 사용하지 않을 속성이 있는 경우 이 경로를 명시적으로 제외해야 합니다.

description 속성을 제외한 모든 경로를 인덱싱하는 인덱싱 정책을 생성합니다.

Azure Portal에서 FoodCollection으로 다시 이동하고 Scale & Settings 링크를 클릭합니다. 
인덱싱 정책 섹션에서 내용을 다음으로 바꿉니다.
```sql
{
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/description/*"
            }
        ]
    }
```
이 새로운 인덱싱 정책은 설명을 제외한 모든 속성에 대해 범위 인덱스를 생성합니다. 
저장을 클릭합니다.   

컨테이너를 다시 인덱싱하는 동안 쓰기 성능은 영향을 받지 않습니다. 그러나 쿼리는 불완전한 결과를 반환할 수 있습니다.   

새 인덱싱 정책을 정의한 후 FoodCollection으로 이동하여 새 SQL 쿼리 추가 아이콘을 선택합니다. 다음 SQL 쿼리를 붙여넣고 쿼리 실행을 선택합니다.   
```sql
SELECT * FROM c WHERE c.manufacturerName = "Kellogg, Co."
```
쿼리 통계 탭으로 이동합니다. manufacturerName 이름이 인덱싱되기 때문에 이 쿼리의 RU 요금이 여전히 낮다는 점을 관찰해야 합니다.   

이제 쿼리 텍스트를 다음으로 바꾸고 쿼리 실행을 선택합니다.   
```sql
SELECT * FROM c WHERE c.description = "Bread, blue corn, somiviki (Hopi)"
```
단일 문서만 반환되는 경우에도 이 쿼리는 RU 요금이 매우 높다는 점을 관찰해야 합니다.   
인덱싱 정책에서 description 속성이 명시적으로 제외되기 때문입니다.   


## 3. Adding a Composite Index
여러 속성을 기준으로 정렬하는 ORDER BY 쿼리의 경우 복합 인덱스가 필요합니다.   
복합 인덱스는 여러 속성에 정의되며 수동으로 만들어야 합니다.   

1. Azure Cosmos DB 블레이드에서 블레이드 왼쪽에 있는 데이터 탐색기(Data Explorer) 링크를 찾아 클릭합니다.
2. 데이터 탐색기 섹션에서 NutritionDatabase 데이터베이스 노드를 확장한 다음 FoodCollection 컨테이너 노드를 확장합니다.
3. 아이콘을 선택하여 새 SQL 쿼리를 추가합니다. 다음 SQL 쿼리를 붙여넣고 쿼리 실행을 선택합니다.
```sql
    SELECT * FROM c WHERE IS_STRING(c.foodGroup) and IS_STRING(c.manufacturerName) ORDER BY c.foodGroup ASC, c.manufacturerName ASC
```

이 쿼리는 다음 오류와 함께 실패합니다.
```
"The order by query does not have a corresponding composite index that it can be served from."
```

하나의 속성으로 ORDER BY 절이 있는 쿼리를 실행하려면 기본 범위 인덱스로 충분합니다.   
ORDER BY 절에 여러 속성이 있는 쿼리에는 복합 인덱스가 필요합니다.   

FoodCollection 노드 내에서 Scale & Settings 링크를 클릭합니다. 
인덱싱 정책 섹션에서 복합 인덱스를 추가합니다.

인덱싱 정책을 다음 텍스트로 바꿉니다.
```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/manufacturerName/*"
        },
        {
            "path": "/foodGroup/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/*"
        },
        {
            "path": "/\"_etag\"/?"
        }
    ],
    "compositeIndexes": [
        [
            {
                "path": "/foodGroup",
                "order": "ascending"
            },
            {
                "path": "/manufacturerName",
                "order": "ascending"
            }
        ]
    ]
}
```
이 새 인덱싱 정책을 저장합니다.    

이 인덱싱 정책은 다음 ORDER BY 쿼리를 허용하는 복합 인덱스를 정의합니다.   
데이터 탐색기의 기존 열려 있는 쿼리 탭에서 실행하여 각각을 테스트합니다.    
복합 인덱스에서 속성의 순서를 정의할 때 ORDER BY 절의 순서와 정확히 일치하거나 모든 경우에 반대 값이어야 합니다.   

```sql
SELECT * FROM c WHERE IS_STRING(c.foodGroup) and IS_STRING(c.manufacturerName) ORDER BY c.foodGroup ASC, c.manufacturerName ASC
```
```sql
SELECT * FROM c WHERE IS_STRING(c.foodGroup) and IS_STRING(c.manufacturerName) ORDER BY c.foodGroup DESC, c.manufacturerName DESC
```   

이제 현재 복합 인덱스가 지원하지 않는 다음 쿼리를 실행해 보십시오.   
```sql
SELECT * FROM c WHERE IS_STRING(c.foodGroup) and IS_STRING(c.manufacturerName) ORDER BY c.foodGroup DESC, c.manufacturerName ASC
```
이 쿼리는 추가 복합 인덱스 없이는 실행되지 않습니다.    
추가 복합 인덱스를 포함하도록 인덱싱 정책을 수정할 수 있습니다.   
```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/manufacturerName/*"
        },
        {
            "path": "/foodGroup/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/*"
        },
        {
            "path": "/\"_etag\"/?"
        }
    ],
    "compositeIndexes": [
        [
            {
                "path": "/foodGroup",
                "order": "ascending"
            },
            {
                "path": "/manufacturerName",
                "order": "ascending"
            }
        ],
        [
            {
                "path": "/foodGroup",
                "order": "descending"
            },
            {
                "path": "/manufacturerName",
                "order": "ascending"
            }
        ]
    ]
}
```
이제 쿼리를 실행할 수 있습니다.    
실습을 완료한 후 [복합 인덱스 정의](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-manage-indexing-policy#composite-indexing-policy-examples)에 대해 자세히 알아볼 수 있습니다.   

<!--
## 4. Adding a spatial index   

### 4.1. Create a new container with volcano data   
먼저 새 데이터베이스 안에 volcanoes 이라는 새 Cosmos 컨테이너를 만듭니다.   
Azure Cosmos DB는 GeoJSON 형식의 데이터 쿼리를 지원합니다.   
이 실습에서는 이 형식으로 지정된 이 컨테이너에 샘플 데이터를 업로드합니다.    
이 volcano.json 샘플 데이터는 기존 영양 데이터 세트보다 지리 공간 쿼리에 더 적합합니다.    
데이터 세트에는 전 세계의 많은 화산에 대한 좌표와 기본 정보가 포함되어 있습니다.     

이 실습에서는 몇 가지 샘플 문서만 업로드하면 됩니다.   
1. Azure Cosmos DB 블레이드에서 데이터 탐색기(Data Explorer) 링크를 찾아 클릭합니다.   
2. 새 컨테이너 추가 아이콘을 선택하십시오.   
![image](https://user-images.githubusercontent.com/44718680/191235306-da8b3a2f-7643-4f6b-ab8c-5812d3aeba8f.png)
3. 컨테이너 생성을 위한 팝업창에 아래와 같이 옵션을 선택하고 생성합니다.   
Database id : VolcanoDatabase    
Container id : VolcanoContainer    
Partition key : /Country   
Container throughput : Manual, 5000   
   <img src="https://user-images.githubusercontent.com/44718680/191235775-a7e2523e-389e-49c6-adae-2cfcf8790b0a.png"  width="500" height="1000"/>

### 4.2. Upload Sample Data   
샘플 데이터를 업로드하면 Azure Cosmos DB는 "Point", "Polygon" 또는 "LineString" 유형의 모든 GeoJSON 데이터에 대한 지리 공간 인덱스를 자동으로 만듭니다.   
1. Azure Portal에서 VolcanoesContainer로 돌아가서 항목 섹션을 클릭합니다.   
2. 업로드 항목 선택   
![image](https://user-images.githubusercontent.com/44718680/191236900-3fbb65de-6698-4255-8770-187bdacc1e83.png)
3. [volcano.json](https://azurecosmosdb.github.io/labs/java/setup/VolcanoData.json) 파일을 다운로드 합니다.



### 4.3. Create geo-spatial indexes in the Volcanoes container
Azure Portal에서 VolcanoesContainer로 다시 이동하고 Scale & Settings 링크를 클릭합니다.    
인덱싱 정책 섹션에서 기존 json 파일을 다음으로 바꿉니다.   

지리 공간 인덱싱은 기본적으로 비활성화되어 있습니다. 
이 인덱싱 정책은 Points, Polygons, MultiPolygon 및 LineStrings를 포함하여 가능한 모든 GeoJSON 유형에 대해 지리 공간 인덱싱을 켭니다. 
범위 인덱스 및 복합 인덱스와 유사하게 지리 공간 인덱스에 대한 정밀도 설정이 없습니다.

[Geo-spatial query 알아보기](https://docs.microsoft.com/en-us/azure/cosmos-db/geospatial#introduction-to-spatial-data) 

### 4.4. Query the Volcano Data
Azure Portal에서 VolcanoesContainer로 다시 이동하고 새 SQL 쿼리를 클릭합니다.    
다음 SQL 쿼리를 붙여넣고 쿼리 실행을 선택합니다.
```sql
SELECT *
FROM volcanoes v
WHERE ST_DISTANCE(v.Location, {
"type": "Point",
"coordinates": [-122.19, 47.36]
}) < 100 * 1000
AND v.Type = "Stratovolcano"
AND v["Last Known Eruption"] = "Last known eruption from 1800-1899, inclusive"
```
이 작업에 대한 쿼리 통계를 관찰합니다. 
컨테이너에 Points에 대한 지리 공간 인덱스가 있기 때문에 이 쿼리는 소량의 RU를 사용했습니다.

### 4.5. Query sample polygon data

```sql 
SELECT *
FROM volcanoes v
WHERE ST_WITHIN(v.Location, {
    "type":"Polygon",
    "coordinates":[[
        [-123.8, 48.8],
        [-123.8, 44.8],
        [-119.8, 44.8],
        [-119.8, 48.8],
        [-123.8, 48.8]
    ]]
    })
```


```sql
SELECT *
FROM volcanoes v
WHERE ST_WITHIN(v.Location, {
    "type":"Polygon",
    "coordinates":[[
        [-123.8, 48.8],
        [-119.8, 48.8],
        [-119.8, 44.8],
        [-123.8, 44.8],
        [-123.8, 48.8]
    ]]
    })
```
-->




Next Lab : [Building a Java Console App on Azure Cosmos DB](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop/blob/main/HandsonLabs/Lab05.md#building-a-java-console-app-on-azure-cosmos-db)   
[목차로 돌아가기](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop#8-java-lab-guides)
