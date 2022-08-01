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
```
git clone https://github.com/AzureCosmosDB/labs.git
```  

## 2. Lab에 사용될 코드 복사
```
cd labs\java\setup

Set-ExecutionPolicy Unrestricted -Scope Process

.\labCodeSetup.ps1
```

## 3. Lab에 사용될 리소스 배포
```
Connect-AzAccount

or

Connect-AzAccount -subscription <subscription id>
```
```powershell
.\labSetup.ps1 -location 'Korea Central' -resourceGroupName 'name' # name을 원하는 이름으로 수정하세요
```

