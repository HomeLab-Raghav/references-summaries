# SQL for SOC Analysts - Practice Exercises
## Hands-On Challenges with Solutions

---

## How to Use This Workbook

1. **Try each exercise yourself first** - Don't look at solutions immediately
2. **Test your queries** - Run them on the chinook.db database
3. **Compare results** - Check if you get the expected output
4. **Review solutions** - Learn alternative approaches
5. **Modify exercises** - Change parameters to practice more

---

## SECTION 1: BASIC QUERIES (Warm-Up)

### Exercise 1.1: Simple SELECT
**Task**: Retrieve the first 10 threat actors from the database.

<details>
<summary>💡 Hint</summary>
Use SELECT * FROM Artist with LIMIT
</details>

<details>
<summary>✅ Solution</summary>

```sql
SELECT * 
FROM Artist 
LIMIT 10;
```

**Expected**: List of threat actor names
</details>

---

### Exercise 1.2: Column Selection
**Task**: Get only the event names and severity IDs from the Track table, limited to 20 rows.

<details>
<summary>✅ Solution</summary>

```sql
SELECT Name, GenreId 
FROM Track 
LIMIT 20;
```
</details>

---

### Exercise 1.3: DISTINCT Values
**Task**: Find all unique companies in the Customer table.

<details>
<summary>✅ Solution</summary>

```sql
SELECT DISTINCT Company 
FROM Customer 
WHERE Company IS NOT NULL;
```
</details>

---

## SECTION 2: FILTERING (WHERE Clause)

### Exercise 2.1: Equality Filter
**Task**: Find all alerts (Invoice) for customer ID 9 (the admin account).

<details>
<summary>✅ Solution</summary>

```sql
SELECT * 
FROM Invoice 
WHERE CustomerId = 9;
```
</details>

---

### Exercise 2.2: Comparison Operators
**Task**: Find all events with more than 5MB of data transferred (Bytes > 5242880).

<details>
<summary>✅ Solution</summary>

```sql
SELECT Name, Bytes 
FROM Track 
WHERE Bytes > 5242880 
ORDER BY Bytes DESC;
```
</details>

---

### Exercise 2.3: AND Condition
**Task**: Find critical (GenreId = 1) events that also transferred more than 1MB.

<details>
<summary>✅ Solution</summary>

```sql
SELECT Name, GenreId, Bytes 
FROM Track 
WHERE GenreId = 1 
  AND Bytes > 1048576 
ORDER BY Bytes DESC;
```
</details>

---

### Exercise 2.4: OR Condition
**Task**: Find events that are either Critical (GenreId = 1) OR High (GenreId = 2) severity.

<details>
<summary>✅ Solution</summary>

```sql
SELECT Name, GenreId 
FROM Track 
WHERE GenreId = 1 OR GenreId = 2 
ORDER BY GenreId;
```
</details>

---

### Exercise 2.5: IN Operator
**Task**: Find events from campaigns 1, 2, or 5.

<details>
<summary>✅ Solution</summary>

```sql
SELECT * 
FROM Track 
WHERE AlbumId IN (1, 2, 5);
```
</details>

---

### Exercise 2.6: BETWEEN
**Task**: Find events with byte size between 1MB and 10MB.

<details>
<summary>✅ Solution</summary>

```sql
SELECT Name, Bytes 
FROM Track 
WHERE Bytes BETWEEN 1048576 AND 10485760;
```
</details>

---

### Exercise 2.7: LIKE Pattern Matching
**Task**: Find all events with "Failed" in their name.

<details>
<summary>✅ Solution</summary>

```sql
SELECT * 
FROM Track 
WHERE Name LIKE '%Failed%';
```
</details>

---

### Exercise 2.8: IS NULL
**Task**: Find all events missing the Composer field.

<details>
<summary>✅ Solution</summary>

```sql
SELECT Name, Composer 
FROM Track 
WHERE Composer IS NULL 
LIMIT 20;
```
</details>

---

## SECTION 3: AGGREGATION FUNCTIONS

### Exercise 3.1: COUNT
**Task**: How many total alerts exist in the database?

<details>
<summary>✅ Solution</summary>

```sql
SELECT COUNT(*) AS TotalAlerts 
FROM Invoice;
```
</details>

---

### Exercise 3.2: COUNT DISTINCT
**Task**: How many unique users have triggered alerts?

<details>
<summary>✅ Solution</summary>

```sql
SELECT COUNT(DISTINCT CustomerId) AS UniqueUsers 
FROM Invoice;
```
</details>

---

### Exercise 3.3: SUM
**Task**: What's the total bytes transferred across all events?

<details>
<summary>✅ Solution</summary>

```sql
SELECT SUM(Bytes) AS TotalBytes 
FROM Track;
```
</details>

---

### Exercise 3.4: AVG
**Task**: What's the average bytes per event?

<details>
<summary>✅ Solution</summary>

```sql
SELECT AVG(Bytes) AS AvgBytes 
FROM Track;
```
</details>

---

### Exercise 3.5: MIN and MAX
**Task**: Find the smallest and largest data transfers.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    MIN(Bytes) AS SmallestTransfer,
    MAX(Bytes) AS LargestTransfer 
FROM Track;
```
</details>

---

## SECTION 4: GROUP BY

### Exercise 4.1: Simple GROUP BY
**Task**: Count how many events exist for each campaign (AlbumId).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    AlbumId,
    COUNT(*) AS EventCount 
FROM Track 
GROUP BY AlbumId 
ORDER BY EventCount DESC;
```
</details>

---

### Exercise 4.2: GROUP BY with Multiple Aggregations
**Task**: For each campaign, show: event count, total bytes, and average bytes.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    AlbumId,
    COUNT(*) AS EventCount,
    SUM(Bytes) AS TotalBytes,
    AVG(Bytes) AS AvgBytes 
FROM Track 
GROUP BY AlbumId 
ORDER BY TotalBytes DESC;
```
</details>

---

### Exercise 4.3: GROUP BY with HAVING
**Task**: Find users with more than 10 alerts.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY CustomerId 
HAVING COUNT(*) > 10 
ORDER BY AlertCount DESC;
```
</details>

---

### Exercise 4.4: Multiple Column GROUP BY
**Task**: Count events grouped by both campaign (AlbumId) and severity (GenreId).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    AlbumId,
    GenreId,
    COUNT(*) AS EventCount 
FROM Track 
GROUP BY AlbumId, GenreId 
ORDER BY AlbumId, GenreId;
```
</details>

---

## SECTION 5: JOINS

### Exercise 5.1: INNER JOIN - Two Tables
**Task**: Get event names with their severity labels (join Track and Genre).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    t.Name AS EventName,
    g.Name AS Severity 
FROM Track t
INNER JOIN Genre g ON t.GenreId = g.GenreId 
LIMIT 20;
```
</details>

---

### Exercise 5.2: Multiple JOINs
**Task**: Show events with their campaign name, threat actor, and severity.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    t.Name AS Event,
    a.Title AS Campaign,
    ar.Name AS ThreatActor,
    g.Name AS Severity 
FROM Track t
JOIN Album a ON t.AlbumId = a.AlbumId
JOIN Artist ar ON a.ArtistId = ar.ArtistId
JOIN Genre g ON t.GenreId = g.GenreId 
LIMIT 20;
```
</details>

---

### Exercise 5.3: LEFT JOIN
**Task**: Show all users with their alert counts (including users with 0 alerts).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    c.CustomerId,
    c.Email,
    COUNT(i.InvoiceId) AS AlertCount 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId 
GROUP BY c.CustomerId, c.Email 
ORDER BY AlertCount DESC;
```
</details>

---

### Exercise 5.4: Find Missing Relations
**Task**: Find users who have NO alerts (using LEFT JOIN and NULL check).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    c.CustomerId,
    c.Email 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId 
WHERE i.InvoiceId IS NULL;
```
</details>

---

## SECTION 6: SUBQUERIES

### Exercise 6.1: Subquery in WHERE
**Task**: Find all events from campaigns by threat actor "APT28".

<details>
<summary>✅ Solution</summary>

```sql
SELECT * 
FROM Track 
WHERE AlbumId IN (
    SELECT AlbumId 
    FROM Album 
    WHERE ArtistId = (
        SELECT ArtistId 
        FROM Artist 
        WHERE Name = 'APT28'
    )
);
```
</details>

---

### Exercise 6.2: Scalar Subquery
**Task**: Find events with above-average byte size.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    Name,
    Bytes,
    (SELECT AVG(Bytes) FROM Track) AS AvgBytes 
FROM Track 
WHERE Bytes > (SELECT AVG(Bytes) FROM Track) 
ORDER BY Bytes DESC;
```
</details>

---

### Exercise 6.3: Derived Table
**Task**: Show user stats (alert count, total severity) for users with > 5 alerts.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    user_stats.*,
    c.Email 
FROM (
    SELECT 
        CustomerId,
        COUNT(*) AS AlertCount,
        SUM(Total) AS TotalSeverity 
    FROM Invoice 
    GROUP BY CustomerId 
    HAVING COUNT(*) > 5
) AS user_stats
JOIN Customer c ON user_stats.CustomerId = c.CustomerId 
ORDER BY AlertCount DESC;
```
</details>

---

## SECTION 7: CASE EXPRESSIONS

### Exercise 7.1: Simple CASE
**Task**: Convert GenreId to severity labels (1=Critical, 2=High, 3=Medium, 4=Low, 5=Info).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    Name,
    GenreId,
    CASE GenreId
        WHEN 1 THEN 'Critical'
        WHEN 2 THEN 'High'
        WHEN 3 THEN 'Medium'
        WHEN 4 THEN 'Low'
        WHEN 5 THEN 'Informational'
        ELSE 'Unknown'
    END AS SeverityLabel 
FROM Track 
LIMIT 20;
```
</details>

---

### Exercise 7.2: Searched CASE
**Task**: Classify events by data size: < 1MB = Small, 1-10MB = Medium, > 10MB = Large.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    Name,
    Bytes,
    CASE 
        WHEN Bytes < 1048576 THEN 'Small'
        WHEN Bytes <= 10485760 THEN 'Medium'
        ELSE 'Large'
    END AS SizeCategory 
FROM Track 
ORDER BY Bytes DESC;
```
</details>

---

### Exercise 7.3: CASE in Aggregation
**Task**: Count events by severity category (Critical, High, Medium, Low).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    SUM(CASE WHEN GenreId = 1 THEN 1 ELSE 0 END) AS Critical,
    SUM(CASE WHEN GenreId = 2 THEN 1 ELSE 0 END) AS High,
    SUM(CASE WHEN GenreId = 3 THEN 1 ELSE 0 END) AS Medium,
    SUM(CASE WHEN GenreId >= 4 THEN 1 ELSE 0 END) AS Low 
FROM Track;
```
</details>

---

## SECTION 8: STRING FUNCTIONS

### Exercise 8.1: LOWER/UPPER
**Task**: Find events with "login" in the name (case-insensitive).

<details>
<summary>✅ Solution</summary>

```sql
SELECT * 
FROM Track 
WHERE LOWER(Name) LIKE '%login%';
```
</details>

---

### Exercise 8.2: LENGTH
**Task**: Find event names longer than 25 characters.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    Name,
    LENGTH(Name) AS NameLength 
FROM Track 
WHERE LENGTH(Name) > 25 
ORDER BY NameLength DESC;
```
</details>

---

### Exercise 8.3: SUBSTR
**Task**: Extract the domain from customer emails.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    Email,
    SUBSTR(Email, INSTR(Email, '@') + 1) AS Domain 
FROM Customer;
```
</details>

---

### Exercise 8.4: CONCATENATION
**Task**: Create full names from FirstName and LastName.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    FirstName || ' ' || LastName AS FullName,
    Email 
FROM Customer;
```
</details>

---

## SECTION 9: DATE & TIME

### Exercise 9.1: Recent Alerts
**Task**: Find alerts from the last 7 days.

<details>
<summary>✅ Solution</summary>

```sql
SELECT * 
FROM Invoice 
WHERE InvoiceDate >= datetime('now', '-7 days') 
ORDER BY InvoiceDate DESC;
```
</details>

---

### Exercise 9.2: Extract Hour
**Task**: Show alert hour distribution.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    strftime('%H', InvoiceDate) AS Hour,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY Hour 
ORDER BY Hour;
```
</details>

---

### Exercise 9.3: After-Hours Detection
**Task**: Find alerts outside business hours (before 8 AM or after 5 PM).

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    InvoiceId,
    InvoiceDate,
    CAST(strftime('%H', InvoiceDate) AS INTEGER) AS Hour 
FROM Invoice 
WHERE CAST(strftime('%H', InvoiceDate) AS INTEGER) NOT BETWEEN 8 AND 17 
ORDER BY InvoiceDate DESC;
```
</details>

---

## SECTION 10: ADVANCED CHALLENGES

### Exercise 10.1: Top 10 High-Risk Users
**Task**: Find users with the most critical/high alerts. Show email, total alerts, and high-severity count.

<details>
<summary>💡 Hint</summary>
Join Customer, Invoice, InvoiceLine, Track. Filter for GenreId <= 2. Group by user.
</details>

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    c.Email,
    COUNT(DISTINCT i.InvoiceId) AS TotalAlerts,
    SUM(CASE WHEN t.GenreId <= 2 THEN 1 ELSE 0 END) AS HighSevAlerts 
FROM Customer c
JOIN Invoice i ON c.CustomerId = i.CustomerId
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId 
GROUP BY c.Email 
HAVING HighSevAlerts > 0 
ORDER BY HighSevAlerts DESC 
LIMIT 10;
```
</details>

---

### Exercise 10.2: Campaign Threat Analysis
**Task**: For each campaign, show: campaign name, threat actor, event count, total bytes, and count of critical events.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    a.Title AS Campaign,
    ar.Name AS ThreatActor,
    COUNT(*) AS Events,
    SUM(t.Bytes) AS TotalBytes,
    SUM(CASE WHEN t.GenreId = 1 THEN 1 ELSE 0 END) AS CriticalEvents 
FROM Track t
JOIN Album a ON t.AlbumId = a.AlbumId
JOIN Artist ar ON a.ArtistId = ar.ArtistId 
GROUP BY a.Title, ar.Name 
ORDER BY CriticalEvents DESC, Events DESC;
```
</details>

---

### Exercise 10.3: User Risk Scoring
**Task**: Create a risk score for each user based on:
- 5 points per critical alert
- 3 points per high alert
- 1 point per other alert
Show users with risk score > 20.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    c.Email,
    COUNT(i.InvoiceId) AS TotalAlerts,
    SUM(CASE 
        WHEN t.GenreId = 1 THEN 5
        WHEN t.GenreId = 2 THEN 3
        ELSE 1
    END) AS RiskScore,
    CASE 
        WHEN SUM(CASE WHEN t.GenreId = 1 THEN 5 WHEN t.GenreId = 2 THEN 3 ELSE 1 END) > 50 THEN 'CRITICAL'
        WHEN SUM(CASE WHEN t.GenreId = 1 THEN 5 WHEN t.GenreId = 2 THEN 3 ELSE 1 END) > 20 THEN 'HIGH'
        ELSE 'MEDIUM'
    END AS RiskLevel 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId
LEFT JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
LEFT JOIN Track t ON il.TrackId = t.TrackId 
GROUP BY c.Email 
HAVING RiskScore > 20 
ORDER BY RiskScore DESC;
```
</details>

---

### Exercise 10.4: Finding Unusual Patterns
**Task**: Find users whose average alert severity is above the global average.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    c.Email,
    COUNT(i.InvoiceId) AS Alerts,
    AVG(i.Total) AS AvgSeverity,
    (SELECT AVG(Total) FROM Invoice) AS GlobalAvg 
FROM Customer c
JOIN Invoice i ON c.CustomerId = i.CustomerId 
GROUP BY c.Email 
HAVING AVG(i.Total) > (SELECT AVG(Total) FROM Invoice) 
ORDER BY AvgSeverity DESC;
```
</details>

---

### Exercise 10.5: Gap Analysis
**Task**: Find campaigns that have events but NO critical severity events.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    a.AlbumId,
    a.Title AS Campaign,
    ar.Name AS ThreatActor 
FROM Album a
JOIN Artist ar ON a.ArtistId = ar.ArtistId 
WHERE EXISTS (
    SELECT 1 FROM Track t WHERE t.AlbumId = a.AlbumId
)
AND NOT EXISTS (
    SELECT 1 FROM Track t WHERE t.AlbumId = a.AlbumId AND t.GenreId = 1
);
```
</details>

---

### Exercise 10.6: Temporal Analysis
**Task**: Show daily alert trends for the last 30 days with severity breakdown.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    date(i.InvoiceDate) AS Date,
    COUNT(DISTINCT i.InvoiceId) AS TotalAlerts,
    SUM(CASE WHEN t.GenreId = 1 THEN 1 ELSE 0 END) AS Critical,
    SUM(CASE WHEN t.GenreId = 2 THEN 1 ELSE 0 END) AS High,
    SUM(CASE WHEN t.GenreId = 3 THEN 1 ELSE 0 END) AS Medium 
FROM Invoice i
LEFT JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
LEFT JOIN Track t ON il.TrackId = t.TrackId 
GROUP BY date(i.InvoiceDate) 
ORDER BY Date DESC 
LIMIT 30;
```
</details>

---

### Exercise 10.7: Correlation Query
**Task**: Find user-campaign pairs with more than 3 events.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    c.Email,
    a.Title AS Campaign,
    COUNT(*) AS EventCount 
FROM Customer c
JOIN Invoice i ON c.CustomerId = i.CustomerId
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId
JOIN Album a ON t.AlbumId = a.AlbumId 
GROUP BY c.Email, a.Title 
HAVING COUNT(*) > 3 
ORDER BY EventCount DESC;
```
</details>

---

## SECTION 11: WINDOW FUNCTION CHALLENGES

### Exercise 11.1: Alert Sequence
**Task**: Number each user's alerts in chronological order.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    InvoiceId,
    CustomerId,
    InvoiceDate,
    ROW_NUMBER() OVER (
        PARTITION BY CustomerId 
        ORDER BY InvoiceDate
    ) AS AlertSequence 
FROM Invoice 
ORDER BY CustomerId, InvoiceDate;
```
</details>

---

### Exercise 11.2: Running Total
**Task**: Calculate running total of bytes transferred.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    TrackId,
    Name,
    Bytes,
    SUM(Bytes) OVER (ORDER BY TrackId) AS CumulativeBytes 
FROM Track 
LIMIT 50;
```
</details>

---

## MASTERY CHALLENGE

### Ultimate SOC Query
**Task**: Create a comprehensive security dashboard query that shows:
1. User email
2. Total alerts
3. Critical alert count
4. High alert count
5. Total bytes transferred
6. Risk level (based on critical/high counts)
7. Last alert timestamp
8. Days since last alert

Only include users with at least 1 alert. Order by risk level and alert count.

<details>
<summary>✅ Solution</summary>

```sql
SELECT 
    c.Email,
    c.Company,
    COUNT(DISTINCT i.InvoiceId) AS TotalAlerts,
    SUM(CASE WHEN t.GenreId = 1 THEN 1 ELSE 0 END) AS CriticalAlerts,
    SUM(CASE WHEN t.GenreId = 2 THEN 1 ELSE 0 END) AS HighAlerts,
    SUM(COALESCE(t.Bytes, 0)) AS TotalBytes,
    CASE 
        WHEN SUM(CASE WHEN t.GenreId = 1 THEN 1 ELSE 0 END) > 5 THEN 'CRITICAL'
        WHEN SUM(CASE WHEN t.GenreId = 2 THEN 1 ELSE 0 END) > 10 THEN 'HIGH'
        WHEN COUNT(DISTINCT i.InvoiceId) > 15 THEN 'MEDIUM'
        ELSE 'LOW'
    END AS RiskLevel,
    MAX(i.InvoiceDate) AS LastAlert,
    CAST((julianday('now') - julianday(MAX(i.InvoiceDate))) AS INTEGER) AS DaysSinceLastAlert 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId
LEFT JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
LEFT JOIN Track t ON il.TrackId = t.TrackId 
GROUP BY c.Email, c.Company 
HAVING COUNT(DISTINCT i.InvoiceId) > 0 
ORDER BY 
    CASE RiskLevel
        WHEN 'CRITICAL' THEN 1
        WHEN 'HIGH' THEN 2
        WHEN 'MEDIUM' THEN 3
        ELSE 4
    END,
    TotalAlerts DESC;
```
</details>

---

## Practice Tips

1. **Start simple** - Master basic exercises before advanced
2. **Type queries** - Don't copy/paste, build muscle memory
3. **Experiment** - Modify queries to see different results
4. **Break down complex queries** - Build step by step
5. **Use EXPLAIN** - Understand query execution
6. **Save your work** - Build a personal query library

---

## Next Steps

After completing these exercises:

1. Review any queries you struggled with
2. Try the Interactive Query Runner (`query_runner.py`)
3. Create your own exercises based on your work environment
4. Practice writing queries without looking at solutions
5. Apply patterns to real security logs

---

**Remember**: Mastery comes from repetition. Work through these exercises multiple times until the patterns become second nature!
