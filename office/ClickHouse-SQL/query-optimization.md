# ClickHouse SQL 查詢優化技巧

## 基本資訊
- **來源**: 布布 research（ClickHouse 官方文件、Altinity 部落格、GitHub 優質範例）
- **年份**: 2024-2026

## 核心內容

### 1. 基礎優化原則

**使用 Projection 和 OrderBy 優化**:
```sql
-- 建立 Projection 加速特定查詢
ALTER TABLE my_table ADD PROJECTION proj_name (
    SELECT * ORDER BY (date, user_id)
);
```

**善用 primary key 排序規則**:
- ClickHouse 按 primary key 排序組織資料
- 將常用於 WHERE 的欄位放在前面
- 範例: `ORDER BY (date, user_id, session_id)`

**避免 SELECT ***:
```sql
-- 不好: 讀取所有欄位
SELECT * FROM events;

-- 好: 只讀取需要的欄位
SELECT user_id, event_type, timestamp FROM events WHERE date = '2024-01-01';
```

### 2. WHERE 子句優化

**使用相同類型比較**:
```sql
-- 不好: 類型轉換
SELECT * FROM t WHERE id = '123';

-- 好: 使用正確類型
SELECT * FROM t WHERE id = 123;
```

**利用 primary key 進行資料剪裁**:
```sql
-- 利用日期分區快速剪裁
SELECT * FROM events 
WHERE date BETWEEN '2024-01-01' AND '2024-01-31'
  AND event_type = 'purchase';
```

**使用 PREWHERE 優化大欄位讀取**:
```sql
-- ClickHouse 自動優化，但可明確指定
SELECT * FROM events PREWHERE event_type = 'click';
```

### 3. 陣列函數優化

**使用 arrayJoin 展開替代 JOIN**:
```sql
-- 展開陣列而非 JOIN 另一張表
SELECT 
    user_id,
    arrayJoin(tags) AS tag
FROM user_events;
```

**ARRAY JOIN 配合 GROUP BY**:
```sql
SELECT 
    tag,
    count() AS cnt
FROM (
    SELECT user_id, arrayJoin(tags) AS tag
    FROM user_events
)
GROUP BY tag
ORDER BY cnt DESC;
```

**arrayFilter 減少資料處理**:
```sql
SELECT arrayFilter(x -> x > 0, [-1, 2, 3, -4, 5]) AS result;
-- 結果: [2, 3, 5]
```

### 4. 索引策略

**使用_skip_indexes 減少資料讀取**:
```sql
-- Bloom filter 索引，加速 IN 子句
ALTER TABLE events ADD INDEX idx_userid user_id TYPE bloom_filter GRANULARITY 1;

-- minmax 索引加速範圍查詢
ALTER TABLE events ADD INDEX idx_date date TYPE minmax GRANULARITY 4;
```

**跳數索引 (Data Skipping Indexes)**:
- `minmax`: 適用於範圍查詢
- `set`: 適用於等值查詢
- `bloom_filter`: 適用於大基數欄位
- `tokenbf_v1`: 適用於文字搜尋
- `ngrambf_v1`: 適用於模糊匹配

### 5. 分割區 (Partitioning) 與分片

**善用 PARTITION BY**:
```sql
CREATE TABLE events (
    date Date,
    user_id UInt32,
    event_type String
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, user_id);
```

**查詢特定分割區**:
```sql
SELECT * FROM events WHERE date >= '2024-06-01' AND date < '2024-07-01';
```

### 6. GROUP BY 與 ORDER BY 優化

**使用 GROUP BY 優化**:
```sql
-- 啟用記憶體優化
SET group_by_two_level_threshold = 1000000;

-- 使用 HAVING 過濾群組結果
SELECT 
    user_id,
    count() AS cnt
FROM events
GROUP BY user_id
HAVING cnt > 100;
```

**LIMIT n BY 取代大量 DISTINCT**:
```sql
-- 每個使用者取最新 10 筆事件
SELECT * 
FROM events 
ORDER BY timestamp DESC 
LIMIT 10 BY user_id;
```

### 7. 物化視圖 (Materialized Views) 優化

**預先彙總加速查詢**:
```sql
CREATE MATERIALIZED VIEW hourly_stats
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, event_type)
AS SELECT 
    toStartOfHour(timestamp) AS hour,
    event_type,
    count() AS cnt,
    sum(amount) AS total_amount
FROM events
GROUP BY hour, event_type;
```

**使用 POPULATE 初始化資料**:
```sql
CREATE MATERIALIZED VIEW hourly_stats
ENGINE = SummingMergeTree()
...
POPULATE AS
SELECT ...;  -- 立即填入現有資料
```

### 8. 常見效能陷阱

| 陷阱 | 解決方案 |
|------|----------|
| 全表掃描 | 使用 WHERE 限制、使用分區和索引 |
| 太多小查詢 | 批次查詢、使用 IN 而非多次查詢 |
| 記憶體不足 | 增加 max_memory_usage、使用樣本查詢 |
| 過多 MERGE 階段 | 減少分割區數量、優化 ORDER BY |

### 9. 設定調優

```sql
-- 調整查詢並行度
SET max_threads = 16;

-- 限制記憶體使用
SET max_memory_usage = 20000000000;  -- 20GB

-- 啟用查詢剖析
SET send_logs_level = 'trace';

-- 遠端查詢超時
SET remote_server_timeout = 60;
```

## 實際應用

### 監控儀表板查詢優化
1. 建立彙總表每小時更新一次
2. 查詢實際資料改為查詢彙總表
3. 使用 PREWHERE 限制不必要的欄位讀取

### 日誌分析效能提升
```sql
-- 原始查詢
SELECT 
    toDate(timestamp) AS date,
    countIf(event_type = 'error') AS errors,
    countIf(event_type = 'click') AS clicks
FROM events
WHERE timestamp >= now() - INTERVAL 7 DAY
GROUP BY date;

-- 優化版本：利用分區和索引
SELECT 
    toDate(timestamp) AS date,
    countIf(event_type = 'error') AS errors,
    countIf(event_type = 'click') AS clicks
FROM events
WHERE date >= today() - 7
  AND timestamp >= today() - INTERVAL 7 DAY
GROUP BY date;
```

## 標籤
#ClickHouse #SQL優化 #查詢優化 #索引 #物化視圖 #MergeTree #資料庫效能