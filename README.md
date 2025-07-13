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

# 📁 프로젝트 개요

| 구분 | 내용 |
|------|------|
| 데이터셋 | Kaggle - Spotify Tracks Attributes and Popularity |
| 데이터 출처 | https://www.kaggle.com/datasets/melissamonfared/spotify-tracks-attributes-and-popularity |
| DB | MySQL (DBeaver) |
| 테이블 규모 | 약 11만 4천 건 |

<br>

---

# 📂 목차

1. [📊 데이터 선정 및 분석](#-데이터-선정-및-분석)
2. [⚙️ 파티션 실행](#️-파티션-실행)
3. [🚀 확장 개념 - 인덱스](#-확장-개념---인덱스 (Index))
4. [🧨 트러블슈팅](#-트러블슈팅)
5. [🪞 회고 및 디벨롭 방향](#-회고-및-디벨롭-방향)

---


# 📊 데이터 선정 및 분석

### 1. 데이터 선정 과정
<br>

<details>
 
<summary><strong> 📁데이터 선정 기준</strong></summary>
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
    RANGE / LIST / HASH 등 다양한 파티셔닝 전략을 진행할 수 있도록 구성
  <br>
</details>
  
  <br>
  
<details>
 
<summary><strong> 📁데이터 레퍼런스 사이트</strong></summary>
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
<summary><strong> 📁데이터 선정 </strong></summary>
  <br>
  
  - **Kaggle - Spotify Tracks Attributes and Popularity** <br>
    https://www.kaggle.com/datasets/melissamonfared/spotify-tracks-attributes-and-popularity
 
 </details>
 
<br><br>
### 2. 데이터 분석

- 🗒️ 주요 컬럼 분석
- 
<img width="621" height="582" alt="image" src="https://github.com/user-attachments/assets/703ea002-33bd-4034-80af-c5950b445c01" />





### 3. 데이터 Import



#### 방법 1 : CREATE TABLE → CSV import [정석]

<details>
   <summary><strong>  📁 TABLE 생성 코드</strong></summary>

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

# ⚙️ 파티션 실행

## ⭐단일 파티션 1 - RANGE 파티션(popularity)

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

#### 🎯 데이터 파티셔닝 전략

- 곡의 `popularity`(인기도) 컬럼을 기준으로 **Range 파티셔닝**을 적용하였습니다.
<br>

#### 💡 파티셔닝 목적

- 인기도 구간(0~100)을 기준으로 곡 데이터를 논리적으로 분할하여, **분석 쿼리의 성능 향상**을 도모합니다. <br>
- 특히 **특정 인기 구간에 대한 빈번한 조회/통계/필터링 작업**이 예상되어, `popularity` 기준의 파티션이 효율적이라고 판단하였습니다.
<br>

### 🧱 파티션 구성 방식

| 파티션 이름   | 포함 구간 (popularity 값) | 설명 |
|----------------|-----------------------------|------|
| `p_0_20`       | 0 이상 21 미만               | 낮은 인기도 곡 |
| `p_21_40`      | 21 이상 41 미만             | 낮은 중간 |
| `p_41_60`      | 41 이상 61 미만             | 평균 수준 |
| `p_61_80`      | 61 이상 81 미만             | 중상위 곡 |
| `p_max`        | 81 이상 101 미만            | 인기 곡 |


→ 총 5개의 파티션으로 구성되어 있으며,  
`popularity` 값에 따라 해당 파티션에 자동으로 분류됩니다.

#### 🔍 활용 예시

- **조회 성능 향상**  
  특정 인기 구간(예: popularity >= 60)만 자주 조회할 경우, 해당 파티션만 접근하게 되어 전체 스캔보다 훨씬 빠른 응답이 가능합니다.

- **범위 기반 통계 분석**  
  구간별 곡 수, 평균 energy 등 인기도별 그룹화 및 통계 분석에 유리합니다.

- **데이터 관리의 용이성**  
  예를 들어 인기도 20 미만 곡이 필요 없어진 경우, `ALTER TABLE DROP PARTITION`으로 빠르게 제거할 수 있습니다.

<br><br>

## ⭐단일 파티션 2 - List파티션 (track_genre)

<details>
<summary>📁 SQL 코드 보기</summary>

 ```
CREATE TABLE spotify_songs_partitioned (
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
  track_genre VARCHAR(50) NOT NULL,
  PRIMARY KEY(id, track_genre)
)
PARTITION BY LIST COLUMNS(track_genre) (
  PARTITION p_pop 			VALUES IN ('pop', 'power-pop', 'pop-film', 'synth-pop', 'party'),
  PARTITION p_rock 			VALUES IN ('rock', 'alt-rock', 'hard-rock', 'punk', 'punk-rock', 'grunge', 'psych-rock', 'rock-n-roll', 'garage', 'indie', 'indie-pop', 'emo', 'guitar', 'rockabilly'),
  PARTITION p_metal 		VALUES IN ('metal', 'heavy-metal', 'death-metal', 'black-metal', 'grindcore', 'metalcore', 'hardcore', 'hardstyle'),
  PARTITION p_electronic 	VALUES IN ('edm', 'electro', 'electronic', 'techno', 'trance', 'house', 'deep-house', 'progressive-house', 'minimal-techno', 'chicago-house', 'detroit-techno', 'dubstep', 'idm', 'drum-and-bass', 'breakbeat', 'club'),
  PARTITION p_hiphop_rnb 	VALUES IN ('hip-hop', 'r-n-b', 'rap', 'funk', 'soul'),
  PARTITION p_jazz_classical VALUES IN ('jazz', 'classical', 'piano', 'instrumental', 'opera'),
  PARTITION p_world 		VALUES IN ('k-pop', 'j-pop', 'j-rock', 'mandopop', 'cantopop', 'french', 'german', 'turkish', 'iranian', 'swedish', 'malay', 'latin', 'latino', 'spanish', 'brazil', 'mpb', 'forro', 'samba', 'pagode', 'sertanejo', 'world-music', 'indian'),
  PARTITION p_country_folk 	VALUES IN ('country', 'bluegrass', 'honky-tonk', 'folk', 'singer-songwriter', 'songwriter'),
  PARTITION p_reggae 		VALUES IN ('reggae', 'reggaeton', 'ska', 'dub', 'dancehall'),
  PARTITION p_child_kids 	VALUES IN ('children', 'kids', 'disney'),
  PARTITION p_ambient_chill VALUES IN ('ambient', 'study', 'sleep', 'chill', 'new-age'),
  PARTITION p_misc 			VALUES IN ('anime', 'blues', 'gospel', 'comedy', 'happy', 'sad', 'romance', 'show-tunes', 'alternative', 'groove', 'goth', 'trip-hop', 'tango', 'acoustic', 'british')
);
```
</details>


#### 🎯 데이터 파티셔닝 전략

- `track_genre` 컬럼을 기준으로 **LIST 파티셔닝**을 적용하여,  
장르별로 데이터를 분산하고 분석 쿼리의 성능을 높이는 구조를 설계하였습니다.
<br>

#### 💡 파티셔닝 목적

- 곡의 장르는 추천 시스템, 사용자 취향 분석, 시장 동향 파악 등에 매우 중요한 요소입니다.
- `track_genre`에 따라 LIST 파티셔닝을 적용함으로써
  - 특정 장르에 대한 조회 성능 향상
  - 장르별 집계/통계의 간소화
  - 데이터 관리(예: 장르별 삭제/이관)의 용이성
  을 기대할 수 있습니다.
<br>

#### 🧱 파티션 구성 예시

| 파티션 이름 | 포함 장르 (예시) |
|-------------|------------------|
| `p_pop`     | pop, power-pop, party 등 |
| `p_rock`    | rock, punk, indie, emo 등 |
| `p_metal`   | metal, hardcore 등 |
| `p_electronic` | edm, techno, house 등 |
| `p_hiphop_rnb` | hip-hop, rap, soul 등 |
| `p_jazz_classical` | jazz, classical, piano 등 |
| `p_world`   | k-pop, j-pop, mandopop, latin 등 |
| `p_country_folk` | country, folk, singer-songwriter 등 |
| `p_reggae`  | reggae, ska, dancehall 등 |
| `p_child_kids` | children, disney 등 |
| `p_ambient_chill` | ambient, sleep, study 등 |
| `p_misc`    | anime, gospel, comedy 등 기타 장르들 |

→ 총 12개의 장르 그룹으로 분리하여,  
  쿼리 수행 시 관련 파티션만을 대상으로 하여 **쿼리 효율 향상**을 도모합니다.


#### 🔍 활용 예시

- 특정 장르 인기곡 조회  
  ```sql
  SELECT track_name, popularity
  FROM spotify_songs_partitioned
  WHERE track_genre = 'rock';
  
<br><br>



## ⭐이중 파티션1 - RANGE + HASH 파티션(popularity, explicit)

<details>
<summary>📁 SQL 코드 보기</summary>

```sql
CREATE TABLE spotify_songs (
  id INT,
  track_id VARCHAR(22),
  artists VARCHAR(600),
  album_name VARCHAR(300),
  track_name VARCHAR(600),
  popularity INT,	-- 0~100까지의 인기도
  duration_ms INT,	-- 트랙 듣는 지속시간
  explicit TINYINT(1) CHECK (explicit IN (0, 1)),  -- BOOLEAN도 가능 / 욕설포함여부
  danceability FLOAT,	-- 댄스에 맞는 곡인지
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

select * from spotify_songs;
drop table spotify_songs;

CREATE TABLE spotify_partitioned (
  id INT,
  track_id VARCHAR(22),
  artists VARCHAR(600),
  album_name VARCHAR(300),
  track_name VARCHAR(600),
  popularity INT,    -- 0~100까지의 인기도
  duration_ms INT,   -- 트랙 듣는 지속시간
  explicit TINYINT(1),-- BOOLEAN도 가능 / 욕설포함여부
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
  PRIMARY KEY (id, popularity, explicit)
)
PARTITION BY RANGE (popularity)
SUBPARTITION BY HASH (explicit)
SUBPARTITIONS 2 (
  PARTITION p_twt VALUES LESS THAN (20),
  PARTITION p_ft VALUES LESS THAN (40),
  PARTITION p_st VALUES LESS THAN (60),
  PARTITION p_et VALUES LESS THAN (80),
  PARTITION p_mx VALUES LESS THAN MAXVALUE
);

```

</details>

#### 🎯 데이터 파티셔닝 전략

- `popularity`(0~100) 컬럼을 기준으로 **RANGE 파티셔닝**,  
- `explicit`(0/1) 컬럼을 기준으로 **HASH 서브파티셔닝(2개)** 을 적용하여 곡 데이터를 분산 저장했습니다.
<br>

#### 💡 파티셔닝 목적

- 곡의 인기도를 구간별로 분리하여 **조회 및 통계 성능 향상**을 유도합니다.
- 각 인기도 구간 안에서 `explicit` 여부에 따라 서브파티션을 분리하여 **욕설 포함 여부에 따른 필터링이나 분석 효율**을 향상시킵니다.
<br>

#### 🧱 파티션 구성 방식

| 파티션 이름 | 인기도 범위 | 서브파티션 수 | 설명 |
|-------------|--------------|----------------|------|
| `p_twt`     | 0~19         | 2              | explicit 여부로 2개 분리 |
| `p_ft`      | 20~39        | 2              | 〃 |
| `p_st`      | 40~59        | 2              | 〃 |
| `p_et`      | 60~79        | 2              | 〃 |
| `p_mx`      | 80~100+      | 2              | 〃 |

→ 총 5개의 popularity 파티션 × 2개의 explicit 서브파티션 = 총 **10개로 분리**


#### 🔍 활용 예시

- **인기도 60 이상 & 욕설 미포함 곡** 조회
  ```sql
  SELECT track_name, popularity
  FROM spotify_partitioned
  WHERE popularity >= 60 AND explicit = 0;

<br><br>
## ⭐이중 파티션2 - LIST + HASH 파티션(track_genre,id)

<details>
<summary>📁 SQL 코드 보기</summary>

```sql
CREATE TABLE spotify_songs_partitioned (
  id INT NOT NULL,
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
  track_genre VARCHAR(50) NOT NULL,
  PRIMARY KEY (id, track_genre)
)
ENGINE = InnoDB
PARTITION BY LIST COLUMNS (track_genre)
SUBPARTITION BY HASH(id)
SUBPARTITIONS 8 (
  PARTITION p_pop VALUES IN ('pop', 'pop-film', 'power-pop', 'indie-pop', 'synth-pop', 'swedish', 'romance'),
  PARTITION p_rock VALUES IN ('rock', 'alt-rock', 'punk', 'punk-rock', 'hard-rock', 'grunge', 'emo', 'rock-n-roll', 'rockabilly', 'psych-rock'),
  PARTITION p_electronic VALUES IN ('edm', 'electronic', 'electro', 'techno', 'trance', 'trip-hop', 'deep-house', 'detroit-techno', 'minimal-techno', 'progressive-house', 'chicago-house', 'idm', 'breakbeat', 'house', 'club'),
  PARTITION p_metal VALUES IN ('metal', 'black-metal', 'death-metal', 'metalcore', 'grindcore', 'heavy-metal', 'hardcore'),
  PARTITION p_hiphoprnb VALUES IN ('hip-hop', 'r-n-b'),
  PARTITION p_jazzbluesclassical VALUES IN ('jazz', 'blues', 'classical', 'piano', 'opera'),
  PARTITION p_latin VALUES IN ('latin', 'latino', 'samba', 'salsa', 'pagode', 'mpb', 'brazil'),
  PARTITION p_worldtraditional VALUES IN ('indian', 'mandopop', 'cantopop', 'malay', 'iranian', 'turkish', 'j-pop', 'j-rock', 'j-idol', 'j-dance', 'anime', 'world-music'),
  PARTITION p_indiefolk VALUES IN ('indie', 'folk', 'singer-songwriter', 'songwriter', 'acoustic', 'guitar'),
  PARTITION p_otherelectronic VALUES IN ('ambient', 'chill', 'dub', 'dubstep', 'garage'),
  PARTITION p_other VALUES IN ('afrobeat', 'bluegrass', 'british', 'children', 'comedy', 'country', 'dance', 'dancehall', 'disco', 'disney', 'drum-and-bass', 'funk', 'french', 'german', 'gospel', 'goth', 'groove', 'happy', 'honky-tonk', 'industrial', 'kids', 'new-age', 'party', 'sad', 'sertanejo', 'show-tunes', 'ska', 'sleep', 'soul', 'spanish', 'study', 'tango')
);

```

</details>

#### 🎯 데이터 파티셔닝 전략

- `track_genre` 컬럼을 기준으로 **LIST 파티셔닝**을 적용하고,  <br>
  각 파티션 내부에 대해 `id` 컬럼을 기준으로 **HASH 서브파티셔닝**을 8개로 구성하였습니다.

<br>

#### 💡 파티셔닝 목적

- 장르별 분석, 필터링, 추천 시스템 등에서 자주 사용되는 `track_genre`를 기준으로 데이터를 분리하여 **조회 효율성을 극대화**
- 각 장르 파티션 내부에서 `id`로 서브파티션을 구성하여 **병렬 처리 및 데이터 분산 저장 최적화**
<br>


#### 🧱 파티션 구성 방식

| 파티션 이름 | 포함 장르 예시 | 서브파티션 수 |
|-------------|----------------|----------------|
| `p_pop`     | pop, indie-pop, romance 등 | 8 |
| `p_rock`    | rock, punk, emo 등 | 8 |
| `p_electronic` | edm, techno, house 등 | 8 |
| `p_metal`   | metal, hardcore 등 | 8 |
| `p_hiphoprnb` | hip-hop, r-n-b | 8 |
| `p_jazzbluesclassical` | jazz, blues, piano 등 | 8 |
| `p_latin`   | latin, samba, mpb 등 | 8 |
| `p_worldtraditional` | k-pop, j-pop, anime 등 | 8 |
| `p_indiefolk` | indie, folk, acoustic 등 | 8 |
| `p_otherelectronic` | ambient, chill, dub 등 | 8 |
| `p_other`   | country, funk, gospel 등 기타 장르 | 8 |

→ 총 **11개의 LIST 파티션 × 8개의 서브파티션 = 88개의 분산 저장 공간**


#### 🔍 활용 예시

- 특정 장르 곡 조회
  ```sql
  SELECT * FROM spotify_songs_partitioned
  WHERE track_genre = 'techno';

<br><br>
---
<br><br>

# 🚀 확장 개념 - 인덱스 (Index)

파티셔닝만으로도 조회 성능을 크게 개선할 수 있지만,  
데이터가 많아질수록 **단순 파티션만으로는 한계에 도달**하게 됩니다.  
그래서 더 나은 성능을 위해 인덱스를 함께 도입하였습니다.

<br>

### 📌 파티셔닝 + 인덱스 = 성능 극대화

- **파티셔닝**은 데이터를 물리적으로 나눠서 불필요한 범위를 건너뛸 수 있게 하고,  
- **인덱스**는 원하는 값을 트리 구조로 빠르게 찾아주는 역할을 합니다.

<br>

둘을 함께 사용하면:

✅ **파티션 프루닝**으로 범위를 좁힌 뒤  
✅ **인덱스를 통해 해당 파티션 내부에서 빠르게 탐색**

> 📈 결과적으로 디스크 I/O를 줄이고, 대용량에서도 **검색 응답 속도가 획기적으로 개선**됩니다.

<br>

### 🌲 인덱스란?

인덱스는 데이터베이스에서 특정 데이터를 빠르게 찾기 위해 **색인**처럼 작동하는 구조입니다.  
MySQL에서는 주로 **B-Tree (또는 B+Tree)** 기반 인덱스를 사용하며, 다음과 같은 구조적 특징이 있습니다:
<br>


| 항목 | 설명 |
|------|------|
| **B-Tree** | 모든 노드에 데이터 존재. 삽입/검색/삭제 성능 일정 |
| **B+Tree** | 실제 데이터는 리프 노드에만 저장. 리프 노드끼리 연결되어 있어 **범위 검색에 유리** |
| **인덱스 생성 위치** | 자주 조회하는 컬럼 (예: `track_genre`, `popularity`, `explicit`) |
| **복합 인덱스** | 두 개 이상의 컬럼에 인덱스를 적용할 수도 있음 (예: `CREATE INDEX idx_genre_pop ON table(track_genre, popularity);`) |

<br>

### 🧪 인덱스 생성 및 비교

- ✅ 인덱스 생성 및 확인 쿼리
  
```sql
-- 인덱스 생성
CREATE INDEX idx_genre_artist ON spotify_songs_partitioned (track_genre, artists);

-- 파티션과 인덱스 키가 올바르게 쓰이고 있는지 확인하는 코드
Explain
SELECT track_name, popularity 
FROM spotify_songs_partitioned
WHERE track_genre = 'pop' artists='Arko' ORDER BY track_name desc limit 10;

```

- ✅ Explain 결과
<img width="734" height="60" alt="image" src="https://github.com/user-attachments/assets/e0aaeef5-e166-472d-80ee-572deb4fae88" />


---


# 💭 인사이트


<details>
<summary><strong>
📁 CPU 시간 비교코드
</strong></summary> 


```sql
-- 1. 프로파일링 기능 켜기
SET profiling = 1;

-- 2. 첫 번째 쿼리 실행
SELECT * FROM spotify
WHERE popularity BETWEEN 61 AND 80;

-- 3. 두 번째 쿼리 실행
SELECT * FROM spotify_partitioned_indexed
WHERE popularity BETWEEN 61 AND 80;

-- 4. 실행된 쿼리 목록 보기
SHOW PROFILES;

-- 5. 각 쿼리의 CPU 시간 상세 보기
SHOW PROFILE CPU FOR QUERY 1;
SHOW PROFILE CPU FOR QUERY 2;
```

</details>

### 📊 실행 시간 vs CPU 시간 비교

#### 🟦 1. 원본 테이블 (`spotify`)

| 단계 | Duration (s) | CPU User (s) | CPU System (s) |
| --- | --- | --- | --- |
| starting | 0.000385 | 0.000379 | 0.000219 |
| executing | 0.005758 | 0.005922 | 0.000000 |
| 기타 단계 합계 | 0.000687 | 0.000632 | 0.000678 |
| **총합** | **0.006830** | **0.006933** | **0.000897** |
|  |  |  |  |

#### 🟩 2. 파티셔닝 테이블 (`spotify_partitioned`)

| 단계 | Duration (s) | CPU User (s) | CPU System (s) |
| --- | --- | --- | --- |
| starting | 0.000096 | 0.000063 | 0.000028 |
| executing | 0.001138 | 0.001139 | 0.000000 |
| 기타 단계 합계 | 0.000246 | 0.000193 | 0.000104 |
| **총합** | **0.001480** | **0.001395** | **0.000132** |



#### 🟨 3. 파티셔닝 + 인덱스 테이블 (`spotify_partitioned_indexed`)

| 단계 | Duration (s) | CPU User (s) | CPU System (s) |
| --- | --- | --- | --- |
| starting | 0.000081 | 0.000053 | 0.000023 |
| executing | 0.001793 | 0.001797 | 0.000000 |
| 기타 단계 합계 | 0.000456 | 0.000379 | 0.000074 |
| **총합** | **0.002330** | **0.002229** | **0.000097** |



| 항목 | 실행 시간 (`Duration`) | CPU 시간 |
| --- | --- | --- |
| 정의 | 쿼리 시작부터 끝까지 걸린 전체 시간 (벽시계 기준) | 실제로 CPU가 연산한 시간 |
| 포함 | 대기 시간, I/O, 락, CPU | 오직 계산/처리한 시간만 |
| 예시 | 10초 (느린 쿼리지만 CPU는 1초만 사용) | 1초 (CPU 효율은 높음) |

✅ 실행 시간은 "사용자가 체감하는 전체 처리 시간"  
✅ CPU 시간은 "CPU가 실제 일한 시간"

<br>

## ✨ 결론


- 🔵 **RANGE 파티셔닝만 적용한 테이블이 가장 빠르고 CPU 사용량도 가장 적음**
  - → 쿼리 조건(`WHERE popularity >= 60`)이 파티션 키에 포함되므로, **파티션 프루닝(Pruning)** 덕분에 검색 범위가 좁아짐

- 🟡 인덱스를 추가했을 경우, 오히려 **CPU 사용량이 증가**
  - → 이유: 파티션 내에서 **인덱스 탐색**이라는 추가 작업이 발생하기 때문
  - → 단, **데이터 양이 매우 클 경우엔** 인덱스 + 파티션 조합이 유리할 수 있음

- ⚪ 원본 테이블은 파티션/인덱스 없이 **풀스캔(Full Table Scan)**
  - → CPU 시간과 실행 시간이 모두 가장 큼



---

# 🧨 트러블슈팅 (Troubleshooting)

데이터 수집 → 테이블 설계 → 파티셔닝까지 진행하는 과정에서 발생한 주요 이슈들과  
그에 대한 해결 과정을 정리했습니다.



## 📥 Import 과정

### (1) CSV 컬럼 타입/크기 미정

<img width="919" height="387" alt="image" src="https://github.com/user-attachments/assets/bd5aa2a8-e546-4a42-ac31-123a5d6cd149" />

- **TROUBLE**  
  CSV 데이터의 각 필드 길이를 알 수 없어 `VARCHAR` 크기 설정에 어려움 발생

- **SOLUTION**  
  ChatGPT에 CSV 파일을 전달해 각 컬럼의 **최대 문자열 길이**를 추출한 후,  
  그 결과를 바탕으로 적절한 `VARCHAR(n)` 크기로 설계


### (2) 예약어 컬럼명 충돌

<img width="731" height="324" alt="image" src="https://github.com/user-attachments/assets/b1665717-4883-4b47-95e2-266fa3709ebb" />=

- **TROUBLE**  
  `INDEX`, `KEY`, `MODE`는 SQL 예약어이므로 컬럼명으로 사용할 수 없음 → 오류 발생

- **SOLUTION**  
  CSV 필드명을 다음과 같이 리네이밍 후 테이블 및 CSV 모두 일치시킴

| 원래 필드명 | 변경 컬럼명 |
|-------------|-------------|
| index       | id          |
| key         | key_col     |
| mode        | mode_col    |


## 🧩 파티셔닝 과정

### (1) HASH 파티셔닝 실패

- **TROUBLE**  
  `track_id`(VARCHAR)를 서브파티션 키로 사용하려고 했을 때 오류 발생  
  > `SQL Error [1491] [HY000]: The SUBPARTITION function returns the wrong type`

- **원인**  
  `HASH 파티셔닝`은 **정수형 컬럼(INT)** 만 사용할 수 있음

- **SOLUTION**  
  `track_id` 대신 `id(INT)` 컬럼을 서브파티션 키로 사용



### (2) LIST 파티셔닝 insert 오류

- **TROUBLE**  
  `track_genre` 기준으로 LIST 파티션을 만들고 insert 시 아래와 같은 오류 발생  
  > `SQL Error [1526] [HY000]: Table has no partition for value from column_list`

- **원인**  
  INSERT하려는 `track_genre` 값이 파티션 정의에 포함되지 않았거나,  
  대/소문자, 공백, 하이픈 등의 차이로 인해 **정의된 값과 불일치**

- **SOLUTION**  
  INSERT 전 `track_genre` 값을 **소문자화 + 공백 제거 + 파티션 조건과 일치하도록 전처리**

<details>
<summary><strong>✅ INSERT 전 정제 쿼리 (클릭해서 열기)</strong></summary>

```sql
SELECT
  id, track_id, artists, album_name, track_name, popularity,
  duration_ms, explicit, danceability, energy, key_col, loudness,
  mode_col, speechiness, acousticness, instrumentalness, liveness,
  valence, tempo, time_signature,
  LOWER(TRIM(track_genre)) AS track_genre
FROM spotify
WHERE track_genre IS NOT NULL
  AND TRIM(track_genre) != ''
  AND LOWER(TRIM(track_genre)) IN (
    'pop', 'power-pop', 'rock', 'edm', 'hip-hop', 'jazz', 'metal',
    'latin', 'ambient', 'children', 'folk', 'punk', 'trance', ...
  );
```
</details>


---
## 🪞 회고 및 디벨롭 방향

## ✅ 프로젝트 회고
<br>

이번 프로젝트를 통해 단순히 파티셔닝을 “이해하는 수준”을 넘어서,  
**직접 실행하고, 성능을 수치로 검증하며, 설계 전략의 중요성**을 체감할 수 있었습니다.
<br><br>

### 🎯 우리가 얻은 인사이트

- 파티셔닝은 단순한 데이터 분산 개념을 넘어 **쿼리 전략에 따라 성능 최적화 효과가 극대화**됨을 확인
- 데이터의 규모나 조건에 따라 **인덱스를 전략적으로 병행 적용**함으로써, 파티셔닝과의 시너지를 도출할 수 있음
- 파티션 키 선정은 단순한 스키마 설계가 아닌, **실제 조회 패턴을 기반으로 한 전략적 결정 요소**임을 체감
- 실습 과정에서 발생한 다양한 오류를 직접 해결하며 **MySQL 파티셔닝의 제약과 적용 가능 범위**에 대한 이해도를 높일 수 있었음
<br>

---

## 😅 아쉬운 점
<br>

- 본 프로젝트는 약 **11만 건 수준의 중소 규모 데이터셋**을 기반으로 실험을 진행  
- 실무에서 접하는 수천만 ~ 수억 건 단위의 **대용량 트래픽/로그/이벤트 데이터**에 비해  
  쿼리 최적화 기법들의 성능 차이가 **뚜렷하게 드러나지 않는 한계**가 있었음

> ✅ 그럼에도 불구하고, 파티셔닝 + 인덱스 조합이 **실제 쿼리 경로에 어떤 영향을 미치는지** 쿼리 실행 계획과 CPU 사용량을 통해 직접 확인한 점은 의미 있는 성과였음

<br>

---

## 🔧 향후 디벨롭 방향

| 개선 포인트 | 내용 |
|-------------|------|
| 📌 더 큰 데이터셋 실험 | 1천만 건 이상 대용량 CSV 수집 및 파티셔닝 성능 실험 |
| 📌 자동화된 전처리 | INSERT 전 `track_genre` 등의 값을 자동 정제할 수 있도록 SQL 함수 or Python 스크립트로 자동화 |
| 📌 성능 로그 수집 자동화 | `SHOW PROFILE` 결과를 시각화하거나 CSV로 export해 성능 비교를 정량화 |
| 📌 외부 시각화 연계 | 파티션별 쿼리 결과를 Python(Pandas/Seaborn)로 시각화하여 효과를 명확히 전달 |





