# 📙 09.10 GCP
## 📊 Big Query
### 간단한 쿼리 작성
```sql
SELECT * 
FROM `billing_dataset.sampleinfotable` 
WHERE Cost > 0
```
- Run 클릭 → Cost가 0보다 큰 행 확인

### 415,602개의 결제 데이터를 조회하는 쿼리 실행
```sql
SELECT 
  billing_account_id, 
  project.id, 
  project.name, 
  service.description, 
  currency, 
  currency_conversion_rate, 
  cost, 
  usage.amount, 
  usage.pricing_unit 
FROM 
  `billing_dataset.sampleinfotable`
```

###  최신 100개의 Cost > 0 데이터 조회
```sql
SELECT 
  service.description, 
  sku.description, 
  location.country, 
  cost, 
  project.id, 
  project.name, 
  currency, 
  currency_conversion_rate, 
  usage.amount, 
  usage.unit 
FROM 
  `billing_dataset.sampleinfotable` 
WHERE Cost > 0 
ORDER BY usage_end_time DESC 
LIMIT 100
```

### Cost가 10달러 이상인 데이터 조회
```sql
SELECT 
  service.description, 
  sku.description, 
  location.country, 
  cost, 
  project.id, 
  project.name, 
  currency, 
  currency_conversion_rate, 
  usage.amount, 
  usage.unit 
FROM 
  `billing_dataset.sampleinfotable` 
WHERE cost > 10
```

### 가장 많은 Billing 기록을 가진 서비스 찾기
```sql
SELECT 
  service.description, 
  COUNT(*) AS billing_records 
FROM 
  `billing_dataset.sampleinfotable` 
GROUP BY service.description 
ORDER BY billing_records DESC
```

### 1달러 초과 청구가 가장 많은 서비스 찾기
```sql
SELECT 
  service.description, 
  COUNT(*) AS billing_records 
FROM 
  `billing_dataset.sampleinfotable` 
WHERE cost > 1 
GROUP BY service.description 
ORDER BY billing_records DESC
```

### 가장 많이 사용된 과금 단위 찾기
```sql
SELECT 
  usage.unit, 
  COUNT(*) AS billing_records 
FROM 
  `billing_dataset.sampleinfotable` 
WHERE cost > 0 
GROUP BY usage.unit 
ORDER BY billing_records DESC
```

### 총 비용이 가장 높은 서비스 찾기
```sql
SELECT 
  service.description, 
  ROUND(SUM(cost),2) AS total_cost 
FROM 
  `billing_dataset.sampleinfotable` 
GROUP BY service.description 
ORDER BY total_cost DESC
```