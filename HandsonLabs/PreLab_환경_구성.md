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
![image](https://user-images.githubusercontent.com/44718680/182066913-759cfdc9-4407-4cf9-a47f-b16d5112a40d.png)



<br></br>
## 3. 리소스 배포 스크립트 수정
1번에서 git clone 한 리포지터리 labs\java\setup\labSetup.ps1 스크립트를 아래 링크의 내용으로 변경   
   
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

![image](https://user-images.githubusercontent.com/44718680/182065234-0ac4af60-7997-44aa-8f43-a1fae288b28a.png)
