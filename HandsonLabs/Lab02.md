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

**데이터베이스와 컨테이너가 생성될 동안 잠시 기다려 주세요**


## 2. Import Lab Data Into Container
Azure Data Factory를 사용하여 Azure Blob Storage의 Nutrition.json 파일에 저장된 JSON 배열 데이터를 가져옵니다.

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






