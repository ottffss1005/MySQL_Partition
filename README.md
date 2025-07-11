#  스포티파이 트랙 데이터를 활용한 Partition Project

<br>

### 👥 Team Members
| 강한솔 | 이정이 | 임채준 | 최소영 |
| :---: | :---: | :---: | :---: |
| [<img width="160px" src="https://github.com/kkangsol.png" />](https://github.com/kkangsol) | [<img width="160px" src="https://github.com/2jeong2.png" />](https://github.com/2jeong2) | [<img width="160px" src="https://github.com/dlacowns21.png" />](https://github.com/dlacowns21) | [<img width="160px" src="https://github.com/ottffss1005.png" />](https://github.com/ottffss1005) |
| [@kkangsol](https://github.com/kkangsol) | [@2jeong2](https://github.com/2jeong2) | [@dlacowns21](https://github.com/dlacowns21) | [@ottffss1005](https://github.com/ottffss1005) |

<br>

# ⚓ 방향성 설정
 ### ❓ 파티션이란 <br>
  논리적으로 하나의 테이블이지만<br>
  물리적으로는 여러개의 파티션으로 나뉘어 데이터들이 각각의 세그먼트에 저장되는 테이블<br>
  
**파티셔닝의 개념을 이론에만 그치지 않고, 실제 성능에 미치는 효과를 정량적으로 분석하고자 하였습니다.** <br>
**나아가 성능 최적화를 위한 확장 기법을 적용하며, 파티션 설계의 전략적 활용 가능성을 실무 관점에서 모색하였습니다.**
<br> <br>

# 🎯 프로젝트 목표
<br>

파티션은 대용량 데이터 처리 환경에서 **성능 최적화를 위한 핵심 전략**으로 작용합니다. <br>
<br>
본 프로젝트에서는 **외부 데이터를 기반으로 파티션을 설계**하고 쿼리 성능 개선을 목적으로 **인덱스 전략을 병행 적용**했습니다. <br>
<br>
이를 통해 **데이터 액세스 범위 축소 및 파티션 프루닝 효과를 직접 검증**하며 파티션이 실질적으로 미치는 영향을 체계적으로 분석하였습니다.

<br>

---

## 📁 프로젝트 개요

| 구분 | 내용 |
|------|------|
| 데이터셋 | Kaggle - Spotify Tracks Attributes and Popularity |
| 데이터 출처 | https://www.kaggle.com/datasets/melissamonfared/spotify-tracks-attributes-and-popularity |
| DB | MySQL (DBeaver) |
| 테이블 규모 | 약 11만 4천 건 |

<br>

---


## 📂 목차

1. [📊 데이터 선정 및 분석](#-데이터-선정-및-분석)
2. [⚙️ 파티션 실행](#️-파티션-실행)
3. [🚀 확장 개념 - 인덱스](#-확장-개념---인덱스)
4. [🧨 트러블슈팅](#-트러블슈팅)
5. [🪞 회고 및 디벨롭 방향](#-회고-및-디벨롭-방향)

---


## 📊 데이터 선정 및 분석

### 1. 데이터 선정 과정
<br>

<details>
 
<summary><strong> 데이터 선정 기준</strong></summary>
  <br>
  
  - **데이터 크기**   <br>
  성능 실험에 유의미한 수준의 데이터량을 갖춘 데이터셋 사용
  <br>
  
  - **정제된 데이터**   <br>
  전처리 부담이 적은 정제된 데이터셋을 사용하여 파티션 과정에 집중할 수 있도록 구성
  <br>
  
  - **다양한 컬럼 수**   <br>
    파티션 기준을 유연하게 설계하고 다양한 방식의 실험 및 최적화가 가능하도록 구성
  <br>
  
  - **다양한 데이터 타입**   <br>
    RANGE / LIST / HASH 등 다양한 파티셔닝 전략을 실습할 수 있도록 구성
  <br>
</details>
  
  <br>
  
<details>
 
<summary><strong> 데이터 레퍼런스 사이트</strong></summary>
  <br>
  
  - **Kaggle** <br>
    https://www.kaggle.com/datasets
  
  - **금융감독원 - 금융상품한눈에** <br>
    https://finlife.fss.or.kr/finlife/api/fncCoApi/list.do?menuNo=700051
     
  - **데이터허브** <br>
    https://datahub.io/

  - **UCL Machine Lerning Repository** <br>
    https://archive.ics.uci.edu/

  - **공공 데이터 포털** <br>
    https://www.data.go.kr/

 </details>
  <br>
  
<details>
<summary><strong> 데이터 선정 </strong></summary>
  <br>
  
  - **Kaggle - Spotify Tracks Attributes and Popularity** <br>
    https://www.kaggle.com/datasets/melissamonfared/spotify-tracks-attributes-and-popularity
 
 </details>
 
<br><br>
### 2. 데이터 분석

- 🗒️ 주요 컬럼 분석
  
<img width="623" height="592" alt="스크린샷 2025-07-12 150548" src="https://github.com/user-attachments/assets/ec86dad2-b03d-493d-87b6-f2e58f2f8bdd" />



### 3. 데이터 임포트



#### 방법 1 : CREATE TABLE → CSV import [정석]

<details>
   <summary><strong>  📄 TABLE 생성 코드</strong></summary>

  <pre><code>
  CREATE TABLE spotify_songs (
  id INT,
  track_id VARCHAR(22),
  artists VARCHAR(600),
  album_name VARCHAR(300),
  track_name VARCHAR(600),
  popularity INT,
  duration_ms INT,
  explicit TINYINT(1) CHECK (explicit IN (0, 1)),  -- BOOLEAN도 가능
  danceability FLOAT,
  energy FLOAT,
  key_col INT,
  loudness FLOAT,
  mode_col INT,
  speechiness FLOAT,
  acousticness FLOAT,
  instrumentalness FLOAT,
  liveness FLOAT,
  valence FLOAT,
  tempo FLOAT,
  time_signature INT,
  track_genre VARCHAR(50)
);
  </code></pre>

</details>
<br>

- 데이터 타입과 제약조건을 명확히 정의할 수 있어 <strong>스키마 일관성과 무결성</strong>을 확보할 수 있음
<br><br>
- 예약어 및 컬럼명 충돌을 사전에 방지할 수 있어 <strong>이식성과 유지보수성</strong>이 향상됨
<br><br>
- Import 과정에서 오류 발생 시 <strong>문제의 원인 추적 및 해결</strong>할 수 있음
<br><br>
- 실무 환경에서는 스키마 설계와 데이터 적재 프로세스를 <strong>명확히 분리</strong>하는 것이 일반적
<br>
<br>

#### 방법 2 : DBeaver 자동 테이블 생성

- 제약조건을 설정할 수 없어, <strong>데이터의 정합성과 무결성을 보장하기 어려움</strong>
<br><br>
- 데이터 설계와 적재가 분리되지 않으면 <strong>데이터 품질 저하, 유지보수의 복잡성 증가, 시스템 안정성 저하</strong> 등의 문제가 발생할 수 있음
<br><br>


---

## ⚙️ 파티션 실행

### 🔹 단일 파티션 1
- 예: `track_genre` 기준 LIST 파티션

### 🔹 단일 파티션 2
- 예: `popularity` 기준 RANGE 파티션

### 🔸 이중 파티션 1
- 예: `popularity` + `explicit` (RANGE + HASH)

### 🔸 이중 파티션 2
- 예: `track_genre` + `id` (LIST + HASH)

---

## 🚀 확장 개념 - 인덱스

- 파티셔닝과 인덱싱 차이
- 인덱스 적용 전후 성능 비교
- 파티션 프루닝 설명

---

## 🧨 트러블슈팅

- 데이터 타입, 예약어 충돌, BOOLEAN 처리 등
- 파티션 조건 미일치 → INSERT 오류
- 파티션 프루닝 안 되는 사례

---

## 🪞 회고 및 디벨롭 방향

- 성능 개선 체감 및 실무적 확장 가능성
- 남은 아쉬운 점 (예: 서브파티션 프루닝 한계, 조건 최적화 등)
- 향후 발전 방향 (→ 날짜 기준 파티션, 로그성 데이터 적용 등)

---




## 🔹 0. 파티션이란? 

<img width="386" height="130" alt="image" src="https://github.com/user-attachments/assets/f1fc510d-f727-48fc-b01c-4f38a2492fcb" />
<br>
- 논리적으로는 하나의 테이블이지만 여러 개의 파티션으로 나뉘어 <br>
  데이터들이 각각의 세그먼트에 저장되는 테이블
- 파티션 키를 PRIMARY KEY 에 포함시켜야하며 파티션 키는 단일 컬럼만 가능하다.<br>
⇒ 유일성 검사 시 어느 파티션을 봐야할 지 모르기때문에 빠른 탐색과 중복 여부 체크를 하기 위해 PRIMARY KEY를 포함시키기 위함 <br>
<br><br>

## 🔹 1. 데이터 수집 & 전처리

- 파이썬으로 데이터 전처리 (문자에 무의미한 끈따옴표 등 삭제)
- SQL 예약어(index, key 등)는 컬럼명(ID, KEY_COL)으로 수정
- 불리언 필드는 0/1로 변환하여 `NUMBER(1)` 또는 `TINYINT(1)`로 매핑
- 필드 최대 길이는 GPT 도구를 활용해 자동 분석하여 설계에 반영
 <br> <br>
### 🛠 기본 과정: 데이터 타입 및 크기 지정

<img width="600" height="260" alt="image (1)" src="https://github.com/user-attachments/assets/17deedac-9625-4725-b483-9add9267381b" />

- DB에 저장하기 전, 각 컬럼의 타입과 최대 길이를 설계
- CSV 데이터 특성상 **길이 예측이 어려워 사전 분석 필요**

#### ⚠ 트러블슈팅

- **TROUBLE**  
  → 데이터 크기가 너무 커서 필드별 최대 길이를 알 수 없음

- **SOLUTION**  
  → ChatGPT에게 CSV 파일을 전달해 각 컬럼의 최대 길이를 자동 추출  
  → 추출된 정보를 바탕으로 `VARCHAR(600)` 등 타입 및 크기 설계

<br>

### 📌 방법 1: TABLE 먼저 생성 후 CSV Import (정석 방식)

<details>
<summary> 🪟TABLE 생성 코드🪟 </summary>

```CREATE TABLE spotify_songs (
  id INT,
  track_id VARCHAR(22),
  artists VARCHAR(600),
  album_name VARCHAR(300),
  track_name VARCHAR(600),
  popularity INT,
  duration_ms INT,
  explicit TINYINT(1) CHECK (explicit IN (0, 1)),  -- BOOLEAN도 가능
  danceability FLOAT,
  energy FLOAT,
  key_col INT,
  loudness FLOAT,
  mode_col INT,
  speechiness FLOAT,
  acousticness FLOAT,
  instrumentalness FLOAT,
  liveness FLOAT,
  valence FLOAT,
  tempo FLOAT,
  time_signature INT,
  track_genre VARCHAR(50)
);
```
</details>

- 컬럼 타입, 제약조건을 **사전에 명확히 통제 가능**
- 예약어 충돌 방지 및 import 실패 시 디버깅 용이
- 실무에서는 **설계 → 적재 분리**가 일반적


<br>

### 📌 방법 2: DBeaver로 CSV Import + 자동 테이블 생성

- 제약조건을 설정할 수 없기 때문에 데이터 정합성 확보가 어려움
- 데이터 설계와 적재가 분리되지 않아 데이터 품질 저하, 유지보수 어려움, 시스템 안정성 저하의 문제가 발생할 수 있음
<br><br>
#### 요약

| 항목 | 정석 방식 | 자동 생성 |
|------|-----------|------------|
| 타입 통제 | 가능 | 불가능 |
| 제약조건 설정 | 가능 | 불가 |
| 실무 사용 | 권장 | 비권장 |
---
<br><br>

#### ⚠️ 트러블슈팅 ⚠️

- **TROUBLE**  
  → 데이터 크기가 너무 커서 필드별 최대 길이를 알 수 없음

- **SOLUTION**  
  → ChatGPT에게 CSV 파일을 전달해 각 컬럼의 최대 길이를 자동 추출  
  → 추출된 정보를 바탕으로 `VARCHAR(600)` 등 타입 및 크기 설계

---


## 📊 Range Partition 결과

---

### ✅ Partition 테이블 생성 및 활용

<details>
<summary>📁 SQL 코드 보기</summary>

```sql
-- 1. 기존 테이블 삭제
DROP TABLE IF EXISTS spotify;

-- 2. 원본 테이블 생성
CREATE TABLE spotify (
  id INT,
  track_id VARCHAR(22),
  artists VARCHAR(600),
  album_name VARCHAR(300),
  track_name VARCHAR(600),
  popularity INT,
  duration_ms INT,
  explicit TINYINT(1),
  danceability FLOAT,
  energy FLOAT,
  key_col INT,
  loudness FLOAT,
  mode_col INT,
  speechiness FLOAT,
  acousticness FLOAT,
  instrumentalness FLOAT,
  liveness FLOAT,
  valence FLOAT,
  tempo FLOAT,
  time_signature INT,
  track_genre VARCHAR(50)
);

-- 3. Range Partition 테이블 생성
CREATE TABLE spotify_partitioned (
  id INT,
  track_id VARCHAR(22),
  artists VARCHAR(600),
  album_name VARCHAR(300),
  track_name VARCHAR(600),
  popularity INT,
  duration_ms INT,
  explicit TINYINT(1),
  danceability FLOAT,
  energy FLOAT,
  key_col INT,
  loudness FLOAT,
  mode_col INT,
  speechiness FLOAT,
  acousticness FLOAT,
  instrumentalness FLOAT,
  liveness FLOAT,
  valence FLOAT,
  tempo FLOAT,
  time_signature INT,
  track_genre VARCHAR(50),
  PRIMARY KEY (id, popularity)
)
PARTITION BY RANGE (popularity) (
  PARTITION p_0_20    VALUES LESS THAN (21),
  PARTITION p_21_40   VALUES LESS THAN (41),
  PARTITION p_41_60   VALUES LESS THAN (61),
  PARTITION p_61_80   VALUES LESS THAN (81),
  PARTITION p_81_100  VALUES LESS THAN (101),
  PARTITION p_max     VALUES LESS THAN MAXVALUE
);

-- 4. 데이터 삽입
INSERT INTO spotify_partitioned
SELECT * FROM spotify;
```

</details>

---

### ⏱️ 쿼리 성능 측정 (실행 시간 기준)

<details>
<summary>📁 SQL 코드 보기</summary>

```sql
-- 실행 시간 측정 (ms 단위)
SET @start_original = CURRENT_TIMESTAMP(3);
SELECT * FROM spotify WHERE popularity BETWEEN 60 AND 80;
SET @end_original = CURRENT_TIMESTAMP(3);

SET @start_partitioned = CURRENT_TIMESTAMP(3);
SELECT * FROM spotify_partitioned WHERE popularity BETWEEN 60 AND 80;
SET @end_partitioned = CURRENT_TIMESTAMP(3);

SET @start_partitioned_indexed = CURRENT_TIMESTAMP(3);
SELECT * FROM spotify_partitioned_indexed WHERE popularity BETWEEN 60 AND 80;
SET @end_partitioned_indexed = CURRENT_TIMESTAMP(3);

-- 실행 시간 출력
SELECT 
  TIMESTAMPDIFF(MICROSECOND, @start_original, @end_original) / 1000 AS `원본_실행시간_ms`,
  TIMESTAMPDIFF(MICROSECOND, @start_partitioned, @end_partitioned) / 1000 AS `RANGE_실행시간_ms`,
  TIMESTAMPDIFF(MICROSECOND, @start_partitioned_indexed, @end_partitioned_indexed) / 1000 AS `RANGE+INDEX_실행시간_ms`;
```

</details>

---

### 🔍 CPU 사용량 비교

<details>
<summary>📁 SQL 프로파일링 코드 보기</summary>

```sql
-- 1. 프로파일링 기능 켜기
SET profiling = 1;

-- 2. 쿼리 실행
SELECT * FROM spotify WHERE popularity BETWEEN 61 AND 80;
SELECT * FROM spotify_partitioned WHERE popularity BETWEEN 61 AND 80;
SELECT * FROM spotify_partitioned_indexed WHERE popularity BETWEEN 61 AND 80;

-- 3. 실행된 쿼리 목록 확인
SHOW PROFILES;

-- 4. CPU 사용 시간 상세 보기
SHOW PROFILE CPU FOR QUERY 1;
SHOW PROFILE CPU FOR QUERY 2;
SHOW PROFILE CPU FOR QUERY 3;
```

</details>

---

### 📊 CPU 사용량 비교 테이블

| 🔍 구분                  | ⏱️ Duration (s) | 🧠 CPU User (s) | ⚙️ CPU System (s) | 🧾 총합 (s)     |
|--------------------------|------------------|------------------|--------------------|------------------|
| 🔵 원본 테이블           | 0.006830         | 0.006933         | 0.000000           | **0.006933**     |
| 🟩 파티셔닝만 적용       | 0.001480         | 0.001395         | 0.000132           | **0.001527**     |
| 🟨 파티셔닝 + 인덱스 적용 | 0.002330         | 0.002229         | 0.000097           | **0.002326**     |

---

### 📌 실행 시간 vs CPU 시간 차이

| 항목       | 실행 시간 (`Duration`)           | CPU 시간 (`CPU_user` + `CPU_system`)     |
|------------|----------------------------------|-------------------------------------------|
| 정의       | 쿼리 실행에 걸린 전체 시간        | 실제 연산에 소요된 CPU 시간                |
| 포함 요소  | I/O, 락, 대기 시간 포함           | 순수 계산 시간                            |
| 중요성     | 사용자 체감 성능 지표             | 서버 자원 최적화 지표                     |

---

### ✅ 결론 및 추천 전략

> **BETWEEN / 범위 조건 쿼리**를 자주 사용하는 경우:

- 🟩 `Range 파티셔닝만 적용`하는 것이 가장 효과적이다.

- 🟨 인덱스는 꼭 필요한 경우에만 고려한다.

> 💡 쿼리 속도뿐만 아니라 **CPU 사용량도 함께 고려**하여 파티셔닝 전략을 수립해야 한다.


