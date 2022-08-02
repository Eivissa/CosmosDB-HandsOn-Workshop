# Importing Data into Azure Cosmos DB with Azure Data Factory
이 실습에서는 Azure에 기본 제공되는 도구를 사용하여 기존 데이터 집합에서 Azure Cosmos DB 컨테이너에 데이터를 채웁니다. 
가져온 후에는 Azure Portal을 사용하여 가져온 데이터를 봅니다.

## 1. Create Azure Cosmos DB Database and Container

1. 리소스 그룹을 선택합니다. 
![image](https://user-images.githubusercontent.com/44718680/182298967-92f8ceb3-773e-48dc-afbb-3c7033bdc07a.png)   

2. Cosmos DB가 생성되어있는 리소스 그룹을 선택 합니다.
![image](https://user-images.githubusercontent.com/44718680/182299036-1d829d6b-ec41-4f63-9a6b-696cbc22cda6.png)   

3. Cosmos DB 리소스를 선택 합니다. 
![image](https://user-images.githubusercontent.com/44718680/182299128-cc7d8aa1-fe5f-4351-b052-4e3431d5a2c9.png)

4. 컨테이너 추가를 선택 합니다. 
![image](https://user-images.githubusercontent.com/44718680/182299322-69d1cd0d-8227-49d8-9edf-ba004e9cf832.png)

### 컨테이너 추가 팝업에서 다음 작업을 수행합니다.

1. 데이터베이스 ID 필드에서 새로 만들기 옵션을 선택하고 ImportDatabase값을 입력합니다.

2. 컨테이너 간 처리량 공유 옵션을 선택하지 마십시오.

3. 데이터베이스에 대한 컨테이너 간 처리량 공유 옵션을 사용하면 해당 데이터베이스에 속한 모든 컨테이너 간에 처리량을 공유할 수 있습니다. Azure Cosmos DB 데이터베이스 내에는 처리량을 공유하는 컨테이너 집합과 전용 처리량이 있는 컨테이너가 있을 수 있습니다.

4. 컨테이너 ID 필드에 FoodCollection값을 입력합니다.

5. 파티션 키 필드에 /foodGroup값을 입력합니다.

6. 컨테이너 처리량(자동 크기 조정) 필드에서 수동 옵션을 선택합니다.

7. 컨테이너 최대 RU/초 필드에 값 11000을 입력합니다.

8. 확인 버튼을 클릭합니다.
