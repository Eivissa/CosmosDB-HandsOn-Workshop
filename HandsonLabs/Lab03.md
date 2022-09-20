# Querying in Azure Cosmos DB
Azure Cosmos DB SQL API 계정은 가장 친숙하고 널리 사용되는 쿼리 언어 중 하나인 SQL(구조적 쿼리 언어)을 JSON 쿼리 언어로 사용하여 항목 쿼리를 지원합니다. 이 랩에서는 Azure Portal을 통해 직접 이러한 풍부한 쿼리 기능을 사용하는 방법을 살펴봅니다. 별도의 도구나 클라이언트 측 코드가 필요하지 않습니다.

## 1. Query Overview
SQL을 사용하여 JSON을 쿼리하면 Azure Cosmos DB에서 레거시 관계형 데이터베이스의 장점을 NoSQL 데이터베이스와 결합할 수 있습니다. 하위 쿼리 또는 집계 함수와 같은 다양한 쿼리 기능을 사용할 수 있지만 NoSQL 데이터베이스에서 데이터 모델링의 많은 이점을 계속 유지할 수 있습니다.

Azure Cosmos DB는 엄격한 JSON 항목만 지원합니다. 유형 시스템 및 표현식은 JSON 유형만 처리하도록 제한됩니다. 자세한 내용은 [JSON 사양](https://www.json.org/)을 참조하세요.

