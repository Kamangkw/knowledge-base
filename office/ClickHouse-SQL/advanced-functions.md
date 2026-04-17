# ClickHouse 進階函數：陣列、JSON 與實用技巧

## 基本資訊
- **來源**: 布布 research（ClickHouse 官方文件、Altinity 技術博客、GitHub 開源專案）
- **年份**: 2024-2026

## 核心內容

### 1. 陣列函數 (Array Functions)

**基礎陣列操作**:
```sql
-- 建立陣列
SELECT array(1, 2, 3, 4, 5) AS arr;

-- 範圍陣列
SELECT range(10);  -- [0,1,2,3,4,5,6,7,8,9]
SELECT range(5, 10);  -- [5,6,7,8,9]

-- 陣列長度
SELECT arrayLength([1, 2, 3]);  -- 3

-- 陣列元素是否存在
SELECT has([1, 2, 3], 2);  -- 1 (true)
SELECT has([1, 2, 3], 5);  -- 0 (false)
```

**陣列聚合函數**:
```sql
-- 陣列求和
SELECT arraySum([1, 2, 3, 4, 5]);  -- 15

-- 陣列乘積
SELECT arrayProduct([1, 2, 3, 4]);  -- 24

-- 陣列平均
SELECT arrayAvg([1, 2, 3, 4]);  -- 2.5
```

**陣列過濾與轉換**:
```sql
-- arrayFilter: 過濾元素
SELECT arrayFilter(x -> x > 2, [1, 2, 3, 4, 5]);  -- [3,4,5]

-- arrayMap: 轉換每個元素
SELECT arrayMap(x -> x * 2, [1, 2, 3]);  -- [2,4,6]

-- 組合使用
SELECT arrayFilter(
    x -> x > 0, 
    arrayMap(x -> x * 2, [-1, 2, -3, 4])
);  -- [4,8]
```

**陣列排序與去重**:
```sql
-- 排序
SELECT arraySort([3, 1, 2]);  -- [1,2,3]
SELECT arrayReverseSort([3, 1, 2]);  -- [3,2,1]

-- 去重
SELECT arrayDistinct([1, 2, 2, 3, 3, 3]);  -- [1,2,3]

-- 陣列拼接
SELECT arrayConcat([1, 2], [3, 4], [5]);  -- [1,2,3,4,5]
```

**arrayJoin: 展開陣列為多列**:
```sql
-- 將陣列展開為單獨的列
SELECT 
    user_id,
    arrayJoin(tags) AS tag
FROM (
    SELECT 1 AS user_id, ['music', 'sports', 'tech'] AS tags
    UNION ALL
    SELECT 2 AS user_id, ['food', 'travel'] AS tags
);
-- 輸出: 
-- 1, music
-- 1, sports
-- 1, tech
-- 2, food
-- 2, travel
```

**陣列切片與查找**:
```sql
-- 切片
SELECT arraySlice([1, 2, 3, 4, 5], 2, 3);  -- [2,3,4] (從索引2開始，取3個)

-- 查找元素索引
SELECT arrayElement([10, 20, 30], 2);  -- 20

-- 查找首次出現索引
SELECT indexOf([1, 2, 3, 2, 1], 2);  -- 2
```

### 2. JSON 提取函數

**JSON 函數家族**:
```sql
-- 解析 JSON 字串
SELECT JSONExtractInt('{"a": 123, "b": 456}', 'a');  -- 123
SELECT JSONExtractString('{"name": "John", "age": 30}', 'name');  -- John

-- JSONExtractRaw: 取得原始 JSON
SELECT JSONExtractRaw('{"arr": [1,2,3]}', 'arr');  -- '[1,2,3]'

-- JSONExtractArray: 解析為陣列
SELECT JSONExtractArray('{"ids": [10, 20, 30]}', 'ids');  -- [10,20,30]
```

**JSON 物件操作**:
```sql
-- 解析整個 JSON 為元組
SELECT JSONExtract('{"name": "John", "age": 30}', 'Tuple(name String, age UInt8)');

-- 解析巢狀 JSON
SELECT 
    JSONExtractInt(json, 'user', 'profile', 'age') AS age
FROM (
    SELECT '{"user": {"profile": {"age": 25}}}' AS json
);
```

**JSON 大家庭 (JSONEachRow)**:
```sql
-- 讀取 NDJSON 格式
SELECT *
FROM file('data.jsonl', JSONEachRow)
WHERE JSONExtractInt(_JSON, 'age') > 18;
```

### 3. 字串函數進階

**字串操控**:
```sql
-- 字串分割
SELECT splitByChar(',', 'a,b,c,d');  -- ['a','b','c','d']

-- 字串替換
SELECT replaceOne('hello world', 'world', 'ClickHouse');  -- 'hello ClickHouse'
SELECT replaceAll('hello world world', 'world', 'CH');  -- 'hello CH CH'

-- 正規表達式
SELECT extract('Order #12345 total: $99.99', '\\d+');  -- '12345'

-- 字串格式化
SELECT format('{1} + {2} = {3}', 1, 2, 3);  -- '1 + 2 = 3'
```

**URL 函數**:
```sql
-- 解析 URL
SELECT extractURLParameter('https://example.com/?name=John&age=30', 'name');  -- 'John'
SELECT URLHierarchy('https://a.b.c/d/e/f');  -- ['https://a.b.c','https://a.b.c/','...']
```

### 4. 日期時間函數

**常用日期函數**:
```sql
-- 日期截斷
SELECT toStartOfHour(now());      -- 小時開始
SELECT toStartOfDay(now());       -- 當天開始
SELECT toStartOfMonth(now());     -- 月初
SELECT toStartOfYear(now());      -- 年初

-- 日期運算
SELECT now() - INTERVAL 7 DAY;    -- 7天前
SELECT toDate('2024-01-15') + INTERVAL 1 MONTH;  -- 2024-02-15

-- 日期萃取
SELECT 
    toYear(date) AS year,
    toMonth(date) AS month,
    toDayOfMonth(date) AS day,
    toHour(timestamp) AS hour
FROM events;
```

**時區處理**:
```sql
-- 轉換時區
SELECT 
    toString(timestamp, 'Asia/Taipei') AS taipei_time,
    toString(timestamp, 'America/New_York') AS ny_time
FROM events;
```

### 5. 條件函數

**if 和多條件**:
```sql
SELECT if(10 > 5, 'Yes', 'No');  -- 'Yes'

-- 巢狀 if
SELECT if(
    score >= 90, 'A',
    if(score >= 80, 'B', 
       if(score >= 70, 'C', 'D'))
) AS grade
FROM students;

-- multiIf: 多條件簡潔寫法
SELECT multiIf(
    score >= 90, 'A',
    score >= 80, 'B',
    score >= 70, 'C',
    'D'
) AS grade;
```

**null 處理**:
```sql
-- coalesce: 取第一個非 null 值
SELECT coalesce(NULL, NULL, 10, 20);  -- 10

-- ifNull: null 替代值
SELECT ifNull(NULL, 'N/A');  -- 'N/A'

-- assumeNotNull: 假設非 null
SELECT assumeNotNull(NULL);  -- 0 或空值（危險操作）
```

### 6. 視窗函數 (Window Functions)

**排名視窗函數**:
```sql
SELECT 
    user_id,
    amount,
    RANK() OVER (PARTITION BY date ORDER BY amount DESC) AS rank,
    ROW_NUMBER() OVER (PARTITION BY date ORDER BY amount DESC) AS row_num,
    DENSE_RANK() OVER (PARTITION BY date ORDER BY amount DESC) AS dense_rank
FROM orders;
```

**聚合視窗函數**:
```sql
SELECT 
    user_id,
    amount,
    SUM(amount) OVER (PARTITION BY user_id) AS user_total,
    AVG(amount) OVER (PARTITION BY user_id) AS user_avg,
    MAX(amount) OVER (PARTITION BY user_id) AS user_max
FROM orders;
```

**LAG/LEAD 取前後值**:
```sql
SELECT 
    date,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY date) AS prev_day_revenue,
    LEAD(revenue, 1) OVER (ORDER BY date) AS next_day_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY date) AS revenue_change
FROM daily_revenue;
```

### 7. 隨機與機率函數

```sql
-- 隨機數
SELECT rand(), rand(42);  -- 隨機種子

-- 隨機陣列
SELECT randomSubset(5, range(10));  -- 從 0-9 中隨機取 5 個

-- 機率分佈
SELECT exponentialDecaySample(0.5, 100);  -- 指數衰減採樣
```

### 8. 實用技巧

**UNEST + GROUP BY 組合**:
```sql
SELECT 
    tag,
    count() AS cnt
FROM (
    SELECT arrayJoin(tags) AS tag
    FROM user_events
)
GROUP BY tag
ORDER BY cnt DESC
LIMIT 10;
```

**CROSS JOIN 配合 arrayJoin**:
```sql
SELECT 
    user_id,
    product_id,
    quantity
FROM users
CROSS JOIN arrayJoin([(1, 10), (2, 20), (3, 30)]) AS t(product_id, quantity);
```

**array join + HAVING**:
```sql
SELECT 
    user_id,
    arrayJoin(interests) AS interest
FROM users
GROUP BY user_id, interest
HAVING interest IN ('AI', 'ML', 'Data Science');
```

## 實際應用

### 使用者行為分析
```sql
-- 找出有特定標籤組合的使用者
SELECT user_id, tags
FROM users
WHERE hasAll(tags, ['AI', 'Machine Learning']);
```

### JSON 日誌解析
```sql
SELECT 
    JSONExtractString(log, 'level') AS level,
    JSONExtractString(log, 'message') AS message,
    count() AS cnt
FROM logs
WHERE JSONExtractString(log, 'level') = 'ERROR'
GROUP BY level, message
ORDER BY cnt DESC;
```

### 時間序列分析
```sql
SELECT 
    toStartOfInterval(timestamp, INTERVAL 1 HOUR) AS hour,
    countIf(event_type = 'click') AS clicks,
    countIf(event_type = 'purchase') AS purchases,
    countIf(event_type = 'purchase') / countIf(event_type = 'click') AS cvr
FROM events
WHERE timestamp >= now() - INTERVAL 24 HOUR
GROUP BY hour
ORDER BY hour;
```

## 標籤
#ClickHouse #陣列函數 #JSON #視窗函數 #日期函數 #字串函數 #SQL技巧