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

![image](https://user-images.githubusercontent.com/44718680/182064874-fe967e8e-87f4-4e70-8364-4593b7ac669b.png)


<br></br>
## 3. 리소스 배포 스크립트 수정
C:\labs\java\setup\labSetup.ps1 스크립트를 아래 링크의 내용으로 변경   
   
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

