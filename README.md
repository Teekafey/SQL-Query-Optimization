# SQL-Query-Optimization


SQL query optimization refers to the process of ensuring your queries are running efficiently, resulting in reduced load times, less resource consumption, and higher performance of your database. Optimizing SQL queries also helps data teams identify and improve poor query performance. It can be an iterative process sometimes.

I was working on optimizing queries and ran into one of the worst SQL queries possible. 

The dataset? **Over 100K rows**.

💭 The goal? Take this query from *__super slow__* to **lightning fast**! ⚡

![The Worst Query Ever (1)](https://github.com/user-attachments/assets/cc50e4c7-284e-47e3-91b3-45f44bb22782)

--------------------------------

## 🚨 Step 1: The “Unoptimized” Query

```SQL
SELECT *
FROM fifa21 a
INNER JOIN [fifa22 b
ON a.RowNum = b.RowNum
WHERE LOWER(a.Nationality) = 'england'
AND a.Club LIKE '%manches%';
```
### ❌ Problems:

SELECT * → Fetching all columns (high I/O).

LOWER(Nationality) = ‘england’ → Breaks indexing, causing a full table scan.

LIKE ‘%manches%’ → Wildcard search prevents index usage, leading to another table scan.

💡 Hint: Table SEEK > Table SCAN

---------------------------------

## 🚀 Step 2: Optimize Column Selection

```SQL
SELECT a.ID, a.Name, a.Age, a.Nationality, a.Club, b.Club AS Fifa22_Club
FROM fifa21 a
INNER JOIN fifa22 b
ON a.RowNum = b.RowNum
WHERE LOWER(a.Nationality) = 'england'
AND a.Club LIKE '%manches%';
```

✅ **Solution:** Select only the necessary columns.

🚀 **Result:** Less memory usage, improved performance.


----------------------------------

## 🔥 Step 3: Fix Function-Based Filtering

```SQL
ALTER TABLE fifa21
ADD Nationality_Lower AS LOWER(Nationality) PERSISTED;
```

```SQL
CREATE INDEX idx_fifa21_nationality ON fifa21(Nationality_Lower);
```

❌ **Problem:** Using LOWER() disables indexes.

✅ **Solution:** Create a computed column and index it.

🚀 **Result:** Faster filtering with an indexed column!

```SQL
-- WHERE CLAUSE NOW LOOKS LIKE THIS

WHERE a.Nationality_Lower = 'england';
```

--------------------------------------

## ⚡ Step 4: Optimize LIKE ‘%manches%’

```SQL
CREATE FULLTEXT INDEX ON fifa21(Club)
KEY INDEX PK_fifa21;
```

```SQL
-- AND CLAUSE NOW LOOKS LIKE THIS

AND a.Club LIKE '%manches%';
```

❌ **Problem:** LIKE ‘%manches%’ forces full table scans.

✅ **Solution:** Use Full-Text Indexing instead.

🚀 **Result:** Index Seek replaces Scan → Faster searches!

---------------------------

## 📈 Step 5: Ensure Efficient JOINs

```SQL
CREATE INDEX idx_fifa21_RowNum ON fifa21(RowNum);
CREATE INDEX idx_fifa22_RowNum ON fifa22(RowNum);
```

❌ **Problem:** Slow JOIN due to missing index.

✅ **Solution:** Index the RowNum column.

🚀 **Result:** JOINs sped up dramatically with Index Seek!

----------------------------

## ⏳ Final Query and Performance Boost:

```SQL
SELECT a.ID, a.Name, a.Age, a.Nationality, a.Club, b.Club AS Fifa22_Club
FROM fifa21 a
INNER JOIN fifa22 b
ON a.RowNum = b.RowNum
WHERE a.Nationality_Lower = 'england'
AND CONTAINS(a.Club, 'manches');
```

🔻 **Before Optimization:** 1.8s (compared to your regular SQL queries).

🔺 **After Optimization:** 0.006s (Way fast compared to the first one 🚀).

### Massive performance gains! 🎯

-------------------------------

**So, there you have it!**

**Thanks for Reading.**
