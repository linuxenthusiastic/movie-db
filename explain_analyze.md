# EXPLAIN ANALYZE Results — Before vs After Indexes

## Query 1: Search by `creation_date`

```sql
SELECT * FROM content.film_work WHERE creation_date = '2020-01-01';
```

### Without Index
```
Seq Scan on film_work (cost=0.00..162.66 rows=8 width=710)
  Filter: (creation_date = '2020-01-01'::date)
  Rows Removed by Filter: 10000
Planning Time: 0.151 ms
Execution Time: 1.391 ms
```

### With Index (`film_work_creation_date_idx`)
```
Bitmap Heap Scan on film_work (cost=4.30..11.62 rows=2 width=84)
  Recheck Cond: (creation_date = '2020-01-01'::date)
  -> Bitmap Index Scan on film_work_creation_date_idx
Planning Time: 0.281 ms
Execution Time: 0.043 ms
```

**Conclusion:** Seq Scan 1.391ms → Bitmap Index Scan 0.043ms. **~32x faster.**

---

## Query 2: Search by `rating`

```sql
SELECT * FROM content.film_work WHERE rating > 8.0;
```

### Without Index
```
Seq Scan on film_work (cost=0.00..162.66 rows=524 width=710)
  Filter: (rating > '8'::double precision)
  Rows Removed by Filter: 8047
Planning Time: 0.072 ms
Execution Time: 8.223 ms
```

### With Index (`film_work_rating_idx`)
```
Bitmap Heap Scan on film_work (cost=39.20..206.27 rows=1925 width=84)
  Recheck Cond: (rating > '8'::double precision)
  Heap Blocks: exact=143
  -> Bitmap Index Scan on film_work_rating_idx
Planning Time: 0.092 ms
Execution Time: 5.078 ms
```

**Conclusion:** Seq Scan 8.223ms → Bitmap Index Scan 5.078ms. **~1.6x faster.**

> Note: Lower improvement because the query returns ~20% of all rows (high selectivity range), making a full scan still competitive. Index shines more on low-selectivity queries.

---

## Query 3: Search by `full_name`

```sql
SELECT * FROM content.person WHERE full_name = 'Person 500';
```

### Without Index
```
Seq Scan on person (cost=0.00..11.75 rows=1 width=548)
  Filter: ((full_name)::text = 'Person 500'::text)
  Rows Removed by Filter: 999
Planning Time: 0.054 ms
Execution Time: 0.132 ms
```

### With Index (`person_full_name_idx`)
```
Index Scan using person_full_name_idx on person
  Index Cond: ((full_name)::text = 'Person 500'::text)
Planning Time: 0.201 ms
Execution Time: 0.037 ms
```

**Conclusion:** Seq Scan 0.132ms → Index Scan 0.037ms. **~3.5x faster.**

---

## Summary

| Query | Without Index | With Index | Improvement |
|-------|--------------|------------|-------------|
| `creation_date = '2020-01-01'` | 1.391 ms (Seq Scan) | 0.043 ms (Bitmap Index Scan) | ~32x faster |
| `rating > 8.0` | 8.223 ms (Seq Scan) | 5.078 ms (Bitmap Index Scan) | ~1.6x faster |
| `full_name = 'Person 500'` | 0.132 ms (Seq Scan) | 0.037 ms (Index Scan) | ~3.5x faster |
