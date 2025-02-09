---
title: Commit 날짜 변경 명령어를 더 간단하게 바꿔보자          
date: 2025-02-09 15:10:58
categories: Git         
published: true 
tags:
- Git  
- Tip      
---  


커밋 날짜를 변경해야 할 때가 가끔 있다. `git commit --date "Wed 29 Jan 2025 00:00:00 KST" -m "커밋메시지"` 이런 식으로 매번 커밋 명령어를 입력하는 건 너무 번거롭다. 좀 더 편하게 바꿀 수 있는 방법을 찾아보았다.   
* 윈도우 환경이므로 Powershell 스크립트로 작성한다.  

### 📌 PowerShell 스크립트 작성  
아래 내용으로 `git-ccommit.ps1` 파일을 만들어 저장한다.  
``` 
param (
    [string]$dateString,
    [string]$message
)

# 날짜 변환 (YYYYMMDD HH:MM:SS → Wed 29 Jan 2025 14:10:58 KST)
$parsedDate = [datetime]::ParseExact($dateString, "yyyyMMdd HH:mm:ss", $null)
$formattedDate = $parsedDate.ToString("ddd dd MMM yyyy HH:mm:ss KST", [System.Globalization.CultureInfo]::InvariantCulture)

# GIT_COMMITTER_DATE 환경 변수 설정 후 git commit 실행
$env:GIT_COMMITTER_DATE = $formattedDate
git commit --date="$formattedDate" -m "$message"

```  

### 📌 PowerShell 실행 권한 설정 (1번만 실행)  
윈도우에서 PowerShell 스크립트를 실행하려면 실행 정책을 변경해두어야 한다. PowerShell을 관리자 권한으로 실행하고 아래 명령어를 입력한다. 

```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser 
``` 
이렇게 하면 로컬에서 만든 스크립트를 실행할 수 있게 된다.  


### 📌 스크립트를 쉽게 실행할 수 있도록 환경 변수 설정
git-ccommit.ps1 파일을 원하는 폴더에 저장해 두고, 이 폴더를 환경 변수(PATH)에 추가해야 어디서든 실행이 가능하다. 

**환경 변수 추가 방법:**  
Win + R → sysdm.cpl → 고급 탭 → 환경 변수 Path에 위에서 정한 폴더 경로를 추가 → PowerShell 재실행

### 📌 실행 결과 확인  
이제 git-ccomit "날짜" -m "메시지" 로 날짜 변경이 가능하다.   
![결과 확인](https://i.imgur.com/exqhRnZ.png)