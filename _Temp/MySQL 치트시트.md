# MySQL 치트시트

## 1. 시간 관련

### 1-1. 시간 관련 함수

 - __현재 날짜/시간 조회__
    - `NOW()`: 현재 날짜 + 시간
    - `CURDATE()`: 현재 날짜
    - `CURTIME()`: 현재 시간
    - `UTC_TIMESTAMP()`: UTC 기준 현재 시간
```sql
-- 2025-11-22 14:30:00
NOW()

-- 2025-11-22
CURDATE()

-- 14:30:00
CURTIME()

-- 2025-11-22 05:30:00 (한국 시간 -9)
UTC_TIMESTAMP()
```
<br/>

 - __시간 차이 계산__
    - `TIMESTAMPDIFF(unit, t1, t2)`: 두 시간의 차이를 UNIT 단위로 반환 (SECOND, MINUTE 등)
    - `TIMEDIFF(t2, t1)`: 시:분:초 형태로 반환
    - `DATEDIFF(d2, d1)`: 날짜 차이만 반환 (일 단위)
```sql
-- TIMESTAMPDIFF(unit, t1, t2)
SELECT TIMESTAMPDIFF(SECOND , '2025-11-22 12:00:00', '2025-11-22 14:00:00'); -- 7200
SELECT TIMESTAMPDIFF(MINUTE , '2025-11-22 12:00:00', '2025-11-22 14:00:00'); -- 120
SELECT TIMESTAMPDIFF(DAY , '2025-11-22 12:00:00', '2025-11-22 14:00:00'); -- 0

-- TIMEDIFF(t2, t1)
-- 미래 시간(t2)을 앞에 배치해야 양수로 계산된다.
SELECT TIMEDIFF('2025-11-21 12:00:00', '2025-11-22 14:00:00'); -- -26:00:00

-- DATEDIFF(d2, d1)
SELECT DATEDIFF('2025-11-21 12:00:00', '2025-11-22 14:00:00'); -- -1
SELECT DATEDIFF('2025-11-21', '2025-11-22'); -- -1
```
<br/>

 - __날짜/시간 더하기/빼기__
    - `DATE_ADD(date, INTERVAL x unit)`: 날짜/시간 더하기	
    - `DATE_SUB(date, INTERVAL x unit)`: 날짜/시간 빼기
```sql
-- 10분 더하기: 2025-11-22 00:10:00
SELECT DATE_ADD('2025-11-22 00:00:00', INTERVAL 10 MINUTE);

-- 1일 더하기: 2025-11-23 00:00:00
SELECT DATE_ADD('2025-11-22 00:00:00', INTERVAL 1 DAY);

-- 3시간 빼기: 2025-11-21 21:00:00
SELECT DATE_ADD('2025-11-22 00:00:00', INTERVAL -3 HOUR);
```
<br/>

 - __시간대 변환__
    - `CONVERT_TZ(datetime, from_tz, to_tz)`: 타임존 변환
```sql
-- 2025-11-22 01:00:00
SELECT CONVERT_TZ('2025-11-22 10:00:00', '+09:00', '+00:00');
```
<br/>

 - __포맷 변경 / 문자열 변환__
    - `DATE_FORMAT(date, '%Y-%m-%d')`: 문자열로 반환
    - `STR_TO_DATE(str, format)`: 문자열 → DATETIME 변환
```sql
-- DATE_FORMAT(date, '%Y-%m-%d')
-- DATETIME -> 문자열: '2025-11-22 10:00:00
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');

-- STR_TO_DATE(str, format)
-- '2025-11-22 10:20:00' 문자열 -> DATETIME
SELECT STR_TO_DATE('2025-11-22 10:20:00', '%Y-%m-%d %H:%i:%s');
```
<br/>

 - __날짜/시간 구성 요소 추출__
    - `YEAR()`: 년도
    - `MONTH()`: 월
    - `DAY()`: 일
    - `HOUR()`: 시
    - `MINUTE()`: 분
    - `SECOND()`: 초
    - `WEEK()`: 주
    - `DAYOFWEEK()`: 요일
    - `DATE()`: 날짜 (YYYY-MM-DD)
    - `TIME()`: 시간 (HH24:MI:SS)
```sql
SELECT YEAR('2025-01-02 10:00:00');     -- 2025
SELECT MONTH('2025-01-02 10:00:00');    -- 1
SELECT DAY('2025-01-02 10:00:00');      -- 2
SELECT HOUR('2025-01-02 10:00:00');     -- 10
SELECT MINUTE('2025-01-02 10:00:00');   -- 0
SELECT SECOND('2025-01-02 10:00:00');   -- 0
SELECT WEEK('2025-01-02 10:00:00');     -- 0: 첫째주
SELECT DAYOFWEEK('2025-01-02 10:00:00');-- 5: 목요일 (1:일, 2:월, 3:화, 4:수, 5:목, 6:금, 7:토)

SELECT DATE('2025-01-02 10:00:00');     -- 2025-01-02
SELECT TIME('2025-01-02 10:00:00');     -- 10:00:00
```
<br/>

 - __DATETIME → UNIX timestamp 변환__
    - `UNIX_TIMESTAMP()`: DATETIME → epoch(sec)
    - `FROM_UNIXTIME()`: epoch(sec) → DATETIME
```sql
-- 1763774400
SELECT UNIX_TIMESTAMP('2025-11-22 10:20:00');

-- '2025-11-22 10:20:00'
SELECT FROM_UNIXTIME(1763774400);
```

### 1-2. 시간 관련 편의 SQL

 - __중간 시간 구하기__
```sql
-- 1. TIMESTAMPDIFF + DATE_ADD
-- TIMESTAMPDIFF(SECOND, startTs, endTs) / 2: 두 시간 차이를 초 단위로 구하고, 절반으로 나누기
-- DATE_ADD(startTs, INTERVAL … SECOND): 시작시간에 절반만큼 더해서 중간 시간을 계산
SELECT DATE_ADD(startTs, INTERVAL TIMESTAMPDIFF(SECOND, startTs, endTs) / 2 SECOND) AS midTime
FROM your_table;

-- 2. TIMESTAMPDIFF + TIMESTAMPADD
SELECT TIMESTAMPADD(SECOND,
                    TIMESTAMPDIFF(SECOND, startTs, endTs) / 2,
                    startTs) AS midTime
FROM your_table;

-- 3. MySQL 8.0 이상
SELECT FROM_UNIXTIME((UNIX_TIMESTAMP(startTs) + UNIX_TIMESTAMP(endTs)) / 2) AS midTime
FROM your_table;
```
<br/>

 - __시간 차이 구하기__
```sql
-- 시간 차이(초): 93600
SELECT TIMESTAMPDIFF(SECOND , '2025-11-21 12:00:00', '2025-11-22 14:00:00');

-- 시간 차이(HH:MI:SS): 단, 24시간이 넘어가는 경우 25 시간으로 누적된다.
-- 26:00:00
SELECT SEC_TO_TIME(TIMESTAMPDIFF(SECOND , '2025-11-21 12:00:00', '2025-11-22 14:00:00'));
```
<br/>

## 2. 문자열 관련 함수

 - `CONCAT(a,b,...)`: 문자열 합침
 - `CONCAT_WS(sep, a,b...)`: 구분자 포함 문자열 합침
 - `LEFT(str, len)`: 왼쪽에서 len 글자
 - `RIGHT(str, len)`: 오른쪽 len 글자
 - `SUBSTRING(str, pos, len)`: 부분 문자열
 - `REPLACE(str, from, to)`: 특정 텍스트 치환
 - `TRIM()`, `LTRIM()`, `RTRIM()`: 공백 제거
 - `LOWER()`, `UPPER()`: 대소문자 변환
 - `LENGTH()`, `CHAR_LENGTH()`: 문자열 길이
```sql
-- Hello World
SELECT CONCAT('Hello ', 'World');

-- Hello World
SELECT CONCAT_WS(' ', 'Hello', 'World');

-- Hello
SELECT LEFT('Hello World', 5);

-- World
SELECT RIGHT('Hello World', 5);

-- SUBSTR / SUBTRING
SELECT SUBSTR('Hello World', 1); -- Hello World: 첫 번째 글자부터
SELECT SUBSTR('Hello World', 6); -- World: 6 번째 글자부터
SELECT SUBSTR('Hello World', 1, 5); -- Hello: 1 번째 글자부터 5개 글자 추출

-- 글자 치환
SELECT REPLACE('Hello World', 'Hello', 'Hi'); -- Hi World

-- 공백 제거
SELECT TRIM(' Hello World '); -- 'Hello World'
SELECT LTRIM(' Hello World '); -- 'Hello World '
SELECT RTRIM(' Hello World '); -- ' Hello World'
```
