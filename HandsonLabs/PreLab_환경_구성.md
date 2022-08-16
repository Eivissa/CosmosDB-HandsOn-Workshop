# Pre-lab: Lab 환경 배포   

<!-- ## 파워쉘 모듈 설치
Azure 파워쉘 모듈이 설치되지 않은 경우 설치 해야 함 
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force
```
Link: https://docs.microsoft.com/en-us/powershell/azure/install-az-ps
-->
## 1.Example Github Workshop resource download   
```cmd
git clone https://github.com/AzureCosmosDB/labs.git
```  
<br></br>
## 2. Lab에 사용될 코드 복사(Powershell)
```powershell
cd labs\java\setup

Set-ExecutionPolicy Unrestricted -Scope Process

.\labCodeSetup.ps1
```
(실행결과)   
<img src="https://user-images.githubusercontent.com/44718680/182066913-759cfdc9-4407-4cf9-a47f-b16d5112a40d.png"  width="600" height="200"/>   



<br></br>
## 3. 리소스 배포 스크립트 수정
1번에서 git clone 한 리포지터리 labs\java\setup\labSetup.ps1 스크립트를 아래 링크의 내용으로 변경   
Azure Powershell에서 지원되지 않는 파라미터의 변경사항을 수정 반영했습니다.   
   
- https://github.com/Eivissa/CosmosDB-HandsOn-Workshop/tree/main/HandsonLabs/Files/labSetup.ps1


<br></br>
## 4. Lab에 사용될 리소스 배포(Powershell)
```powershell
Connect-AzAccount

or

Connect-AzAccount -subscription <subscription id>
```
```powershell
.\labSetup.ps1 -location 'Korea Central' -resourceGroupName 'name' # name을 원하는 이름으로 수정하세요
```
<br></br>
## 5. Azure 배포 결과 확인
1. http://portal.azure.com 로그인 및 올바른 구독인지 확인   
2. 리소스 그룹(Resource Group) 클릭    
![image](https://user-images.githubusercontent.com/44718680/182079888-16604b53-8567-4ebd-8d50-2540878cc68a.png)   
3. Cosmos DB가 배포된 리소스 그룹 클릭   
![image](https://user-images.githubusercontent.com/44718680/182079982-cce35025-be62-4dff-807d-05c320b886d3.png)   
4. Cosmos DB 리소스 클릭   
![image](https://user-images.githubusercontent.com/44718680/182080100-05b2c10b-b593-45dc-9a0e-8cb56a5b22d0.png)   
5. Cosmos DB 설정의 키 메뉴     
![image](https://user-images.githubusercontent.com/44718680/182080253-284c303e-a2ca-4ea8-b754-cb4fc76fab8b.png)   
6. Cosmos DB Key 정보 확인
![image](https://user-images.githubusercontent.com/44718680/182080319-8bda1aef-1a1e-4c10-bec6-e83a1327ab1b.png)
 
Next Lab : [Creating a container in Azure Cosmos DB](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop/blob/main/HandsonLabs/Lab01.md#creating-a-container-in-azure-cosmos-db)   
[목차로 돌아가기](https://github.com/Eivissa/CosmosDB-HandsOn-Workshop#8-java-lab-guides)
