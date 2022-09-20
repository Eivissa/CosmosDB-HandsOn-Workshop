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
몇 가지 간단한 예를 살펴보겠습니다(Azure Portal에 입력할 필요가 없으며 여기에서 검토할 수 있습니다).   

FoodCollection 내에서 문서에는 다음 스키마가 있습니다(간단성을 위해 일부 속성이 제거됨).   
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
? 문자는 Azure Cosmos DB가 이 노드를 넘어서는 더 이상 경로를 인덱싱하지 않아야 함을 나타냅니다. 위의 예에서 NutritionValue 아래에는 추가 경로가 없습니다.   
문서를 수정하고 여기에 경로를 추가하는 경우 위의 예에서 와일드카드 문자 '\*'를 사용하면 이름을 명시적으로 언급하지 않고도 속성이 인덱싱됩니다.  




