# 차트 SQL 치트시트

## 10분 단위로 변환

```sql
-- 10분 단위 그루핑
-- reg_dttm을 10분 단위로 변환하고, 그루핑 진행
-- 온도: 평균값 사용
-- 사용시간: 합계값 사용
SELECT TRUNCATE(AVG(A.humidity), 0) AS humidity,
       SUM(A.usage_time) AS usage_time,
       A.reg_dttm
FROM (
    SELECT humidity, 
           usage_time,
           FROM_UNIXTIME(
            FLOOR(UNIX_TIMESTAMP(reg_dttm) / 600) * 600 -- N * 60: N분 단위 그룹
        ) AS reg_dttm
    FROM device_data A
    WHERE device_id = '1' AND day_date = '2026-02-17'
    ) A
GROUP BY A.reg_dttm
ORDER BY A.reg_dttm;
```



