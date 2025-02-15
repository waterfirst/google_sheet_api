
# YouTube 댓글 수집 및 워드클라우드 생성 매뉴얼

이 매뉴얼은 특정 YouTube 동영상의 댓글을 Google Sheets로 가져오고, 이를 활용하여 워드클라우드를 생성하는 과정을 단계별로 안내합니다. 이 과정을 통해 데이터 수집부터 시각화까지 원활하게 수행할 수 있습니다.

---

## 목차

1. [개요](#1-개요)
2. [YouTube Data API 설정](#2-youtube-data-api-설정)
   - [2.1 Google Cloud 프로젝트 생성 또는 선택](#21-google-cloud-프로젝트-생성-또는-선택)
   - [2.2 YouTube Data API 활성화](#22-youtube-data-api-활성화)
   - [2.3 API 키 발급 및 설정](#23-api-키-발급-및-설정)
3. [Google Sheets에서 댓글 가져오기](#3-google-sheets에서-댓글-가져오기)
   - [3.1 Google Sheets 준비](#31-google-sheets-준비)
   - [3.2 Apps Script 작성 및 실행](#32-apps-script-작성-및-실행)
4. [워드클라우드 생성](#4-워드클라우드-생성)
   - [방법 1: Google Sheets 애드온 사용](#방법-1-google-sheets-애드온-사용)
   - [방법 2: Google Apps Script 및 외부 서비스 활용](#방법-2-google-apps-script-및-외부-서비스-활용)
5. [추가 팁 및 고려사항](#5-추가-팁-및-고려사항)
6. [문제 해결](#6-문제-해결)
7. [결론](#7-결론)

---

## 1. 개요

이 매뉴얼은 다음 두 가지 주요 작업을 수행하는 방법을 설명합니다:

1. **YouTube 댓글 수집:** 특정 YouTube 동영상의 댓글을 Google Sheets로 가져옵니다.
2. **워드클라우드 생성:** 수집된 댓글을 바탕으로 워드클라우드를 생성하여 시각화합니다.

이를 위해 YouTube Data API, Google Sheets, Google Apps Script, 그리고 워드클라우드 생성 도구를 활용합니다.

---

## 2. YouTube Data API 설정

YouTube 댓글을 수집하기 위해서는 YouTube Data API를 활성화하고 API 키를 발급받아야 합니다.

### 2.1 Google Cloud 프로젝트 생성 또는 선택

1. **Google Cloud Console 접속:**
   - [Google Cloud Console](https://console.cloud.google.com/)에 접속하여 Google 계정으로 로그인합니다.

2. **프로젝트 선택 또는 생성:**
   - 상단 메뉴에서 **프로젝트 선택** 드롭다운을 클릭합니다.
   - 기존 프로젝트를 선택하거나, **새 프로젝트** 버튼을 클릭하여 새 프로젝트를 생성합니다.
   - 새 프로젝트를 생성할 경우, 프로젝트 이름을 입력하고 **만들기**를 클릭합니다.

### 2.2 YouTube Data API 활성화

1. **API 및 서비스 라이브러리로 이동:**
   - 왼쪽 사이드바에서 **API 및 서비스** > **라이브러리**를 클릭합니다.

2. **YouTube Data API v3 검색 및 활성화:**
   - 검색창에 **"YouTube Data API v3"**를 입력하고 검색합니다.
   - **"YouTube Data API v3"**를 클릭한 후 **"사용"** 버튼을 클릭하여 API를 활성화합니다.

### 2.3 API 키 발급 및 설정

1. **사용자 인증 정보로 이동:**
   - **API 및 서비스** > **사용자 인증 정보**로 이동합니다.

2. **API 키 생성:**
   - 상단의 **"사용자 인증 정보 만들기"** 버튼을 클릭하고 **"API 키"**를 선택합니다.
   - 생성된 API 키가 팝업으로 표시됩니다. **"복사"** 버튼을 클릭하여 API 키를 안전한 곳에 저장합니다.

3. **API 키 제한 설정 (선택 사항):**
   - 생성된 API 키 옆의 **"편집"** 아이콘을 클릭합니다.
   - **"애플리케이션 제한"**과 **"API 제한"**이 올바르게 설정되었는지 확인합니다.
     - **애플리케이션 제한:** 기본적으로 제한을 해제하거나, Apps Script에서 사용할 경우 **"없음"**으로 설정하는 것이 간단합니다.
     - **API 제한:** **"YouTube Data API v3"**가 허용되어 있는지 확인합니다.
   - **"저장"** 버튼을 클릭하여 변경 사항을 저장합니다.

> **주의:** API 키는 민감한 정보이므로 외부에 노출되지 않도록 안전하게 관리하세요.

---

## 3. Google Sheets에서 댓글 가져오기

YouTube Data API를 활용하여 특정 동영상의 댓글을 Google Sheets로 가져옵니다.

### 3.1 Google Sheets 준비

1. **Google Sheets 열기:**
   - [Google Sheets](https://sheets.google.com/)에 접속하여 새 스프레드시트를 생성하거나 기존 스프레드시트를 엽니다.

2. **시트 구조 설정:**
   - 첫 번째 시트에 다음과 같은 헤더를 추가합니다:
     - **A1:** `작성자`
     - **B1:** `댓글`
     - **C1:** `좋아요 수`
     - **D1:** `작성일`

### 3.2 Apps Script 작성 및 실행

1. **Apps Script 편집기 열기:**
   - 상단 메뉴에서 **"확장 프로그램"** > **"Apps Script"**를 클릭합니다.

2. **스크립트 작성:**
   - 기본으로 생성된 코드를 모두 삭제하고, 아래의 간단한 스크립트를 복사하여 붙여넣습니다.

   ```javascript
   /**
    * 특정 YouTube 동영상의 댓글을 가져와 Google Sheets에 입력하는 함수
    */
   function fetchYouTubeComments() {
     const apiKey = 'YOUR_YOUTUBE_API_KEY'; // 발급받은 API 키로 교체하세요
     const videoUrl = 'https://www.youtube.com/watch?v=Vyuu-ufQJ7M'; // 댓글을 가져올 유튜브 동영상 URL
     
     // URL에서 비디오 ID 추출
     const videoIdMatch = videoUrl.match(/v=([^&]+)/);
     if (!videoIdMatch || videoIdMatch.length < 2) {
       SpreadsheetApp.getUi().alert('유효하지 않은 YouTube URL입니다.');
       return;
     }
     const videoId = videoIdMatch[1];
     
     const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
     sheet.clearContents(); // 기존 내용 삭제 (원하는 경우)
     // 헤더 설정
     sheet.appendRow(['작성자', '댓글', '좋아요 수', '작성일']);
     
     let nextPageToken = '';
     let row = 2;
     const maxResults = 100; // 한 번에 가져올 댓글 수 (최대 100)
     
     try {
       const apiUrl = `https://www.googleapis.com/youtube/v3/commentThreads?part=snippet&videoId=${videoId}&maxResults=${maxResults}&key=${apiKey}`;
       
       const response = UrlFetchApp.fetch(apiUrl);
       const data = JSON.parse(response.getContentText());
       
       if (data.items) {
         data.items.forEach(item => {
           const comment = item.snippet.topLevelComment.snippet;
           sheet.getRange(row, 1).setValue(comment.authorDisplayName);
           sheet.getRange(row, 2).setValue(comment.textDisplay);
           sheet.getRange(row, 3).setValue(comment.likeCount);
           sheet.getRange(row, 4).setValue(comment.publishedAt);
           row++;
         });
       } else {
         SpreadsheetApp.getUi().alert('댓글이 없습니다.');
       }
       
       SpreadsheetApp.getUi().alert('댓글이 성공적으로 가져와졌습니다!');
     } catch (error) {
       Logger.log('오류 발생: ' + error);
       SpreadsheetApp.getUi().alert('오류가 발생했습니다: ' + error.message);
     }
   }
   ```

   > **주의:** `'YOUR_YOUTUBE_API_KEY'` 부분을 실제 발급받은 YouTube Data API 키로 반드시 교체하세요.
   >
   > 예:
   >
   > ```javascript
   > const apiKey = 'AIza...'; // 실제 API 키
   > ```

3. **스크립트 저장:**
   - 왼쪽 상단의 **"저장"** 아이콘(💾)을 클릭하거나 `Ctrl + S`를 눌러 스크립트를 저장합니다.
   - 파일 이름을 묻는다면 `Code.gs` 또는 원하는 이름으로 설정할 수 있습니다.

4. **스크립트 실행:**
   - **함수 선택:** 스크립트 편집기 상단의 **드롭다운 메뉴**에서 `fetchYouTubeComments`를 선택합니다.
   - **실행 버튼 클릭:** ▶️(실행) 버튼을 클릭하여 함수를 실행합니다.

5. **권한 부여:**
   - 스크립트를 처음 실행할 때, Google 계정의 권한을 승인해야 합니다.
   - **"권한 검토"** 버튼을 클릭합니다.
   - 나타나는 창에서 **"앱 이름을 확인할 수 없음"**을 클릭한 후 **"프로젝트(비공개 앱)으로 이동"**을 선택합니다.
   - 필요한 권한을 부여합니다.

6. **실행 결과 확인:**
   - 스크립트가 성공적으로 실행되면 Google Sheets로 돌아가 A2:D열에 댓글 데이터가 입력된 것을 확인할 수 있습니다.

---

## 4. 워드클라우드 생성

Google Sheets에 수집된 댓글을 활용하여 워드클라우드를 생성하는 다양한 방법을 소개합니다.

### 방법 1: Google Sheets 애드온 사용

가장 간단한 방법은 Google Sheets용 워드클라우드 애드온을 사용하는 것입니다. **Word Cloud Generator** 애드온을 예로 들어 설명합니다.

#### 1단계: 워드클라우드 애드온 설치

1. **Google Sheets 문서 열기:**
   - 이미 댓글이 저장된 Google Sheets 문서를 엽니다.

2. **애드온 설치:**
   - 상단 메뉴에서 **"확장 프로그램"** > **"애드온 가져오기"**를 클릭합니다.
   - 검색창에 **"Word Cloud"**를 입력하고 **"Word Cloud Generator"** 또는 유사한 애드온을 선택합니다.
   - **"설치"** 버튼을 클릭하고, 필요한 권한을 승인하여 애드온을 설치합니다.

> **참고:** 다양한 워드클라우드 애드온이 있으니, 평점과 리뷰를 참고하여 적합한 것을 선택하세요.

#### 2단계: 데이터 준비

1. **댓글 데이터 확인:**
   - 댓글이 **B** 열에 저장되어 있다고 가정합니다 (예: B2:B100).

2. **불필요한 텍스트 정리:**
   - 워드클라우드를 생성하기 전에, 불필요한 특수 문자나 중복되는 공백을 제거하는 것이 좋습니다.
   - 예를 들어, C 열에 정리된 댓글을 생성할 수 있습니다:
     ```plaintext
     C2: =TRIM(REGEXREPLACE(B2, "[^가-힣a-zA-Z0-9\s]", ""))
     ```
   - 이 공식을 C2에 입력한 후, 아래로 드래그하여 모든 댓글에 적용합니다.
   - 이 공식은 한글, 영어, 숫자 및 공백을 제외한 모든 특수 문자를 제거합니다.

#### 3단계: 워드클라우드 생성

1. **애드온 실행:**
   - 상단 메뉴에서 **"확장 프로그램"** > **"Word Cloud Generator"** > **"Start"**를 클릭하여 애드온을 실행합니다.

2. **데이터 선택:**
   - 애드온 창에서 워드클라우드를 생성할 데이터 범위를 선택합니다. 예를 들어, **C2:C100** 범위를 선택합니다.

3. **워드클라우드 설정:**
   - **언어 설정:** 한국어 워드클라우드를 생성하기 위해 **한국어**를 선택합니다.
   - **불용어(Stop Words) 설정:** 불용어(자주 등장하지만 분석에 불필요한 단어)를 추가로 설정할 수 있습니다. 예를 들어, "그리고", "하지만" 같은 단어 대신 한국어의 불용어를 추가하세요.
   - **단어 개수:** 워드클라우드에 포함할 단어의 개수를 설정합니다. 예를 들어, 상위 100개 단어.

4. **워드클라우드 생성:**
   - 설정을 마친 후 **"Generate"** 버튼을 클릭합니다.
   - 생성된 워드클라우드 이미지를 Google Sheets 내에 삽입하거나, 이미지 파일로 저장할 수 있습니다.

#### 장단점

- **장점:**
  - 간편함: 몇 번의 클릭만으로 워드클라우드를 생성할 수 있습니다.
  - 시각적 효과: 다양한 색상과 형태로 워드클라우드를 시각화할 수 있습니다.

- **단점:**
  - 맞춤화 제한: 일부 애드온은 워드클라우드의 세부 설정이 제한적일 수 있습니다.

### 방법 2: Google Apps Script 및 외부 서비스 활용

애드온을 사용하지 않고, Google Apps Script를 통해 데이터를 처리한 후 외부 워드클라우드 생성 도구를 사용하는 방법입니다.

#### 1단계: 데이터 전처리 (단어 빈도 계산)

1. **Apps Script 편집기 열기:**
   - 상단 메뉴에서 **"확장 프로그램"** > **"Apps Script"**를 클릭합니다.

2. **단어 빈도 계산 스크립트 작성:**
   - 아래의 스크립트를 복사하여 붙여넣습니다.

   ```javascript
   /**
    * YouTube 댓글을 기반으로 단어 빈도를 계산하여 워드클라우드용 데이터를 생성하는 함수
    */
   function generateWordFrequency() {
     const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
     const comments = sheet.getRange('C2:C' + sheet.getLastRow()).getValues();
     
     const wordCounts = {};
     const stopWords = ['그리고', '하지만', '그런데', '그러나', '이것', '저것', '것', '그것', '나는', '저는', '그는', '그녀는']; // 필요한 불용어 추가
     
     comments.forEach(row => {
       const comment = row[0];
       if (comment) {
         const words = comment.split(/\s+/);
         words.forEach(word => {
           const cleanWord = word.replace(/[^가-힣a-zA-Z0-9]/g, '').trim(); // 특수 문자 제거
           if (cleanWord.length > 1 && !stopWords.includes(cleanWord)) { // 단어 길이 및 불용어 필터링
             wordCounts[cleanWord] = (wordCounts[cleanWord] || 0) + 1;
           }
         });
       }
     });
     
     // 결과를 시트에 기록
     const resultSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('WordFrequency') || SpreadsheetApp.getActiveSpreadsheet().insertSheet('WordFrequency');
     resultSheet.clearContents();
     resultSheet.appendRow(['단어', '빈도']);
     
     for (const [word, count] of Object.entries(wordCounts)) {
       resultSheet.appendRow([word, count]);
     }
     
     SpreadsheetApp.getUi().alert('단어 빈도 계산이 완료되었습니다!');
   }
   ```

   - **스크립트 설명:**
     - 댓글 데이터를 읽어 단어를 추출하고, 불용어 및 단어 길이를 필터링하여 단어 빈도를 계산합니다.
     - 결과는 **WordFrequency**라는 새 시트에 저장됩니다.

3. **스크립트 저장 및 실행:**
   - 스크립트를 저장하고, `generateWordFrequency` 함수를 실행합니다.
   - **권한 부여:** 스크립트 실행 시 필요한 권한을 승인합니다.

4. **단어 빈도 데이터 확인:**
   - **WordFrequency** 시트에 단어와 그 빈도가 기록된 것을 확인할 수 있습니다.

#### 2단계: 외부 워드클라우드 생성 도구 사용

1. **WordFrequency 데이터 복사:**
   - **WordFrequency** 시트에서 **A2:B** 범위 (단어와 빈도) 전체를 선택하고 복사(Ctrl + C)합니다.

2. **WordClouds.com 접속:**
   - 웹 브라우저에서 [WordClouds.com](https://www.wordclouds.com/)에 접속합니다.

3. **데이터 붙여넣기:**
   - **"Word list"** 버튼을 클릭합니다.
   - **"Words"** 입력창에 복사한 데이터를 붙여넣기(Ctrl + V) 합니다.
   - **"Number of Words"**를 원하는 개수로 설정할 수 있습니다.

4. **워드클라우드 생성:**
   - **"Apply"** 버튼을 클릭하여 워드클라우드를 생성합니다.
   - 생성된 워드클라우드를 다운로드하거나 공유할 수 있습니다.

#### 장단점

- **장점:**
  - 유연성: 데이터 처리를 스크립트로 세밀하게 제어할 수 있습니다.
  - 맞춤화: 외부 도구의 다양한 설정을 활용하여 워드클라우드를 맞춤 설정할 수 있습니다.

- **단점:**
  - 복잡성: 애드온에 비해 몇 가지 단계를 더 거쳐야 합니다.
  - 외부 도구 의존: 외부 서비스를 사용해야 하므로 인터넷 연결과 도구의 사용 방법을 숙지해야 합니다.

---

## 5. 추가 팁 및 고려사항

### 1. 쿼터 관리

- **YouTube Data API의 쿼터 제한:**
  - YouTube Data API는 일일 및 분당 쿼터 제한이 있습니다.
  - 한 번에 최대 100개의 댓글을 가져오기 때문에, 단순한 작업에는 문제가 없겠지만 빈번한 실행 시 쿼터를 초과할 수 있습니다.

### 2. API 키 보안 강화

- **API 키 노출 방지:**
  - 스크립트 내에 API 키를 직접 입력하지 않고, **Google Apps Script의 Properties Service**를 사용하여 안전하게 관리할 수 있습니다.

  **방법:**

  1. **API 키 저장 함수 추가:**

     ```javascript
     function setYouTubeApiKey() {
       const apiKey = 'YOUR_YOUTUBE_API_KEY'; // 실제 API 키로 교체하세요
       PropertiesService.getScriptProperties().setProperty('YOUTUBE_API_KEY', apiKey);
     }
     ```

     - `setYouTubeApiKey` 함수를 실행하여 API 키를 저장합니다.

  2. **API 키 불러오기:**

     ```javascript
     function fetchYouTubeComments() {
       const apiKey = PropertiesService.getScriptProperties().getProperty('YOUTUBE_API_KEY');
       // 나머지 코드 동일...
     }
     ```

     - 기존 `const apiKey = 'YOUR_YOUTUBE_API_KEY';` 부분을 위 코드로 교체합니다.
     - 이렇게 하면 코드 내에 API 키가 노출되지 않습니다.

### 3. 자동화 및 트리거 설정

- **정기적인 업데이트:**
  - 댓글을 정기적으로 업데이트하고 싶다면, Apps Script의 **트리거** 기능을 활용할 수 있습니다.

  **설정 방법:**

  1. **트리거 추가:**
     - Apps Script 편집기에서 왼쪽 메뉴의 **"트리거"** 아이콘(시계 모양)을 클릭합니다.
     - **"트리거 추가"** 버튼을 클릭합니다.

  2. **트리거 설정:**
     - **"실행할 함수 선택"**: `fetchYouTubeComments`
     - **"배포 대상"**: Head
     - **"실행할 이벤트 소스"**: 시간 기반
     - **"타입"**: 예를 들어, "매일"을 선택하고 원하는 시간을 설정합니다.
     - **"저장"** 버튼을 클릭합니다.

  3. **트리거 확인:**
     - 설정된 트리거가 목록에 추가된 것을 확인할 수 있습니다.

  > **주의:** 트리거를 사용할 경우, API 쿼터를 초과하지 않도록 주의하세요.

### 4. 오류 발생 시 디버깅

- **로그 확인:**
  - Apps Script 편집기에서 상단 메뉴의 **"보기"** > **"실행 로그"**를 클릭하여 실행 로그를 확인할 수 있습니다.
  - 로그를 통해 API 요청 URL, 응답 코드, 응답 내용을 확인하여 문제를 파악할 수 있습니다.

- **예외 처리 강화:**
  - 스크립트에 `try-catch` 블록을 추가하여 예외 발생 시 로그에 기록하거나 사용자에게 알림을 보낼 수 있습니다.

  ```javascript
  try {
    // 기존 코드
  } catch (error) {
    Logger.log('에러 발생: ' + error);
    SpreadsheetApp.getUi().alert('에러가 발생했습니다: ' + error.message);
  }
  ```

### 5. 데이터 전처리 및 품질 향상

- **불용어 제거:**
  - 자주 등장하지만 의미 없는 단어(예: "그리고", "하지만")를 제거하여 핵심 단어의 가시성을 높입니다.

- **단어 길이 필터링:**
  - 너무 짧거나 의미 없는 단어를 제거합니다. 예를 들어, 한 글자 단어는 제외할 수 있습니다.

- **동의어 통합:**
  - 다양한 형태의 단어를 통합하여 분석합니다. 예를 들어, "좋다", "좋아"를 모두 "좋다"로 통일할 수 있습니다.

- **대소문자 통일:**
  - 대소문자 차이를 없애기 위해 모든 단어를 소문자 또는 대문자로 변환합니다.

---

## 6. 문제 해결

### YouTube Data API 403 오류 발생

403 오류는 주로 API 접근 권한 문제나 설정 불일치로 인해 발생합니다. 다음 단계를 통해 문제를 해결할 수 있습니다.

1. **YouTube Data API 활성화 확인:**
   - Google Cloud Console에서 프로젝트가 올바른지 확인하고, YouTube Data API가 활성화되어 있는지 다시 한 번 확인합니다.

2. **API 키 확인 및 설정:**
   - API 키가 정확한지, 그리고 제한이 올바르게 설정되었는지 확인합니다.
   - **애플리케이션 제한:** **"없음"**으로 설정하거나, 필요에 따라 적절히 설정합니다.
   - **API 제한:** **"YouTube Data API v3"**가 허용되어 있는지 확인합니다.

3. **쿼터 확인:**
   - **API 및 서비스** > **대시보드** > **YouTube Data API v3**에서 할당된 쿼터와 사용량을 확인합니다.
   - 쿼터를 초과했다면, 쿼터를 추가로 신청하거나, 일부 요청을 줄입니다.

4. **새 API 키 생성 및 테스트:**
   - 기존 API 키에 문제가 있을 수 있으므로, 새로운 API 키를 생성하여 시도해봅니다.
   - 스크립트에서 새로운 API 키로 교체하고 다시 실행해봅니다.

5. **서비스 상태 확인:**
   - [Google Cloud Status Dashboard](https://status.cloud.google.com/)에서 YouTube Data API v3의 서비스 상태를 확인합니다.

6. **추가 도움 받기:**
   - Google Cloud 지원팀에 문의하거나, [Stack Overflow](https://stackoverflow.com/questions/tagged/google-apps-script) 등 커뮤니티 포럼에서 비슷한 문제를 검색하거나 질문을 남깁니다.

---

## 7. 결론

이 매뉴얼을 통해 특정 YouTube 동영상의 댓글을 Google Sheets로 수집하고, 이를 활용하여 워드클라우드를 생성하는 방법을 단계별로 이해하고 실습할 수 있습니다. **Google Sheets 애드온을 사용하는 방법**이 가장 간단하며, **Google Apps Script와 외부 도구를 활용하는 방법**은 더 높은 유연성과 맞춤화를 제공합니다.

성공적인 데이터 수집 및 시각화를 기원하며, 추가적인 질문이나 도움이 필요하시면 언제든지 문의해주세요!

