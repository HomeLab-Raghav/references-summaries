# SQL for SOC Analysts: Complete Guide Using Chinook Database
## From Basic to Advanced - Every Query Type You'll Need

---

## TABLE OF CONTENTS
1. [Data Retrieval (SELECT)](#1-data-retrieval)
2. [Filtering Rows (WHERE)](#2-filtering-rows)
3. [Sorting and Limiting](#3-sorting-and-limiting)
4. [Aggregation Functions](#4-aggregation-functions)
5. [GROUP BY and HAVING](#5-group-by-and-having)
6. [JOINs](#6-joins)
7. [Subqueries](#7-subqueries)
8. [CASE Expressions](#8-case-expressions)
9. [String Functions](#9-string-functions)
10. [Date & Time Functions](#10-date-time-functions)
11. [Mathematical & Logical](#11-mathematical-logical)
12. [UNION Operations](#12-union-operations)
13. [EXISTS / NOT EXISTS](#13-exists-not-exists)
14. [Window Functions](#14-window-functions)
15. [Schema Discovery](#15-schema-discovery)

---

## DATABASE SCHEMA OVERVIEW

**Chinook Database adapted for SOC Analysis:**

- **Artist** → Threat Actors (APT groups, threat actors)
- **Album** → Campaigns (attack campaigns)
- **Track** → Events/Logs (individual security events)
- **MediaType** → Event Types (network, auth, file access, etc.)
- **Genre** → Severity Levels (Critical, High, Medium, Low, Info)
- **Customer** → Users (system users)
- **Employee** → SOC Analysts
- **Invoice** → Alerts (security alerts)
- **InvoiceLine** → Alert Details (events in an alert)
- **Playlist** → Watchlists (threat hunting lists)
- **PlaylistTrack** → Watchlist Items

---

## 1. DATA RETRIEVAL (SELECT)

### 1.1 SELECT * (Get All Columns)
**Use Case:** Quick view of all alert data

```sql
-- View all alerts
SELECT * FROM Invoice LIMIT 5;

-- View all threat actors
SELECT * FROM Artist;
```

**SOC Scenario:** First look at an alert queue

---

### 1.2 SELECT Specific Columns
**Use Case:** Focus on key fields for triage

```sql
-- Get alert ID, user, and timestamp
SELECT InvoiceId, CustomerId, InvoiceDate 
FROM Invoice;

-- Get event names and severity
SELECT Name, GenreId 
FROM Track;
```

**SOC Scenario:** Extracting only relevant fields for dashboard

---

### 1.3 SELECT Multiple Columns
**Use Case:** Correlation of related fields

```sql
-- User activity summary
SELECT CustomerId, FirstName, LastName, Email, Company
FROM Customer;

-- Event details for investigation
SELECT TrackId, Name, AlbumId, GenreId, Milliseconds, Bytes
FROM Track;
```

---

### 1.4 SELECT DISTINCT (Unique Values)
**Use Case:** Finding unique indicators

```sql
-- How many unique users generated alerts?
SELECT DISTINCT CustomerId 
FROM Invoice;

-- What unique event types occurred?
SELECT DISTINCT Name 
FROM MediaType;

-- Which threat actors are active?
SELECT DISTINCT Name 
FROM Artist;
```

**SOC Scenario:** Identifying scope of compromise

---

### 1.5 SELECT with Aliases (AS)
**Use Case:** Readable output for reports

```sql
-- Rename columns for clarity
SELECT 
    InvoiceId AS AlertID,
    CustomerId AS UserID,
    InvoiceDate AS Timestamp,
    Total AS Severity
FROM Invoice;

-- Threat actor campaigns
SELECT 
    ar.Name AS ThreatActor,
    al.Title AS Campaign
FROM Artist ar
JOIN Album al ON ar.ArtistId = al.ArtistId;
```

**SOC Scenario:** Creating executive-ready reports

---

### 1.6 SELECT DISTINCT with Alias
**Use Case:** Clean, unique reporting

```sql
-- Unique companies under attack
SELECT DISTINCT Company AS TargetOrganization
FROM Customer
WHERE Company IS NOT NULL;

-- Unique severity levels seen
SELECT DISTINCT Name AS SeverityLevel
FROM Genre;
```

---

## 2. FILTERING ROWS (WHERE CLAUSE)

### 2.1 Comparison Operators

#### Equal (=)
```sql
-- All alerts for specific user
SELECT * FROM Invoice 
WHERE CustomerId = 1;

-- All critical severity events
SELECT * FROM Track 
WHERE GenreId = 1;  -- 1 = Critical
```

**SOC Scenario:** Investigating specific user's alerts

---

#### Not Equal (!= or <>)
```sql
-- All non-informational events
SELECT * FROM Track 
WHERE GenreId != 5;  -- 5 = Informational

-- Alerts not from user 1
SELECT * FROM Invoice 
WHERE CustomerId <> 1;
```

---

#### Less Than / Greater Than (<, >)
```sql
-- Large data transfers (> 5MB)
SELECT Name, Bytes 
FROM Track 
WHERE Bytes > 5242880;

-- Events lasting less than 10 seconds
SELECT Name, Milliseconds 
FROM Track 
WHERE Milliseconds < 10000;
```

**SOC Scenario:** Finding data exfiltration attempts

---

#### Less/Greater Than or Equal (<=, >=)
```sql
-- Medium severity and above
SELECT * FROM Track 
WHERE GenreId <= 3;  -- 1=Critical, 2=High, 3=Medium

-- Events with significant duration
SELECT Name, Milliseconds 
FROM Track 
WHERE Milliseconds >= 60000;
```

---

### 2.2 Logical Operators

#### AND (All conditions must be true)
```sql
-- Critical events from specific campaign
SELECT * FROM Track 
WHERE GenreId = 1 AND AlbumId = 2;

-- Large file transfers that are critical
SELECT Name, Bytes, GenreId 
FROM Track 
WHERE Bytes > 1048576 AND GenreId <= 2;

-- Alerts from specific user in last period
SELECT * FROM Invoice 
WHERE CustomerId = 9 
  AND InvoiceDate >= '2025-01-15';
```

**SOC Scenario:** Narrowing investigation to specific threat

---

#### OR (Any condition can be true)
```sql
-- Critical OR High severity events
SELECT * FROM Track 
WHERE GenreId = 1 OR GenreId = 2;

-- Events from multiple campaigns
SELECT * FROM Track 
WHERE AlbumId = 1 OR AlbumId = 2 OR AlbumId = 3;

-- Alerts from suspicious users
SELECT * FROM Invoice 
WHERE CustomerId = 9 OR CustomerId = 10;
```

---

#### NOT (Negation)
```sql
-- Everything except informational
SELECT * FROM Track 
WHERE NOT GenreId = 5;

-- Users who are not from TechCorp
SELECT FirstName, LastName, Company 
FROM Customer 
WHERE NOT Company = 'TechCorp';
```

---

#### Complex Combinations
```sql
-- Critical OR High events from specific campaigns
SELECT * FROM Track 
WHERE (GenreId = 1 OR GenreId = 2) 
  AND (AlbumId = 1 OR AlbumId = 2);

-- Large transfers that aren't informational
SELECT Name, Bytes, GenreId 
FROM Track 
WHERE Bytes > 5242880 
  AND NOT GenreId = 5;
```

**SOC Scenario:** Building complex detection logic

---

### 2.3 Special Filters

#### IN (Match any value in list)
```sql
-- Events from multiple specific campaigns
SELECT * FROM Track 
WHERE AlbumId IN (1, 2, 5, 7);

-- Critical and High severity only
SELECT * FROM Track 
WHERE GenreId IN (1, 2);

-- Specific users of interest
SELECT * FROM Customer 
WHERE CustomerId IN (1, 5, 9, 10);
```

**SOC Scenario:** Checking against IOC list

---

#### NOT IN (Exclude values)
```sql
-- Exclude low-priority events
SELECT * FROM Track 
WHERE GenreId NOT IN (4, 5);  -- Exclude Low and Info

-- Events not from known-good campaigns
SELECT * FROM Track 
WHERE AlbumId NOT IN (7, 11, 12);
```

---

#### BETWEEN (Range checking)
```sql
-- Events within byte size range
SELECT Name, Bytes 
FROM Track 
WHERE Bytes BETWEEN 1048576 AND 10485760;

-- Medium-severity range
SELECT * FROM Track 
WHERE GenreId BETWEEN 2 AND 4;

-- Alerts in date range
SELECT * FROM Invoice 
WHERE InvoiceDate BETWEEN '2025-01-01' AND '2025-01-31';
```

**SOC Scenario:** Finding anomalies within normal ranges

---

#### LIKE (Pattern matching)
```sql
-- Find login-related events
SELECT * FROM Track 
WHERE Name LIKE '%Login%';

-- Find events with 'Failed' in name
SELECT * FROM Track 
WHERE Name LIKE '%Failed%';

-- Events starting with 'File'
SELECT * FROM Track 
WHERE Name LIKE 'File%';

-- Events ending with 'Detected'
SELECT * FROM Track 
WHERE Name LIKE '%Detected';

-- User emails from specific domain
SELECT * FROM Customer 
WHERE Email LIKE '%@techcorp.com';
```

**SOC Scenario:** Searching logs for keywords

---

#### NOT LIKE (Pattern exclusion)
```sql
-- Exclude successful logins
SELECT * FROM Track 
WHERE Name NOT LIKE '%Successful%';

-- Non-techcorp users
SELECT * FROM Customer 
WHERE Email NOT LIKE '%@techcorp.com';
```

---

#### Wildcards
```sql
-- % = any number of characters
SELECT * FROM Track WHERE Name LIKE 'P%';  -- Starts with P
SELECT * FROM Track WHERE Name LIKE '%Attack%';  -- Contains Attack

-- _ = single character
SELECT * FROM Artist WHERE Name LIKE 'APT__';  -- APT + 2 chars (APT28, APT29)
```

---

#### IS NULL / IS NOT NULL
```sql
-- Events without composer (missing data)
SELECT * FROM Track 
WHERE Composer IS NULL;

-- Customers with companies
SELECT * FROM Customer 
WHERE Company IS NOT NULL;

-- Alerts without billing info (incomplete)
SELECT * FROM Invoice 
WHERE BillingAddress IS NULL;
```

**SOC Scenario:** Finding gaps in telemetry

---

## 3. SORTING AND LIMITING OUTPUT

### 3.1 ORDER BY ASC (Ascending)
```sql
-- Events ordered by size (smallest first)
SELECT Name, Bytes 
FROM Track 
ORDER BY Bytes ASC;

-- Alerts by date (oldest first)
SELECT InvoiceId, InvoiceDate 
FROM Invoice 
ORDER BY InvoiceDate ASC;
```

---

### 3.2 ORDER BY DESC (Descending)
```sql
-- Largest data transfers first (potential exfil)
SELECT Name, Bytes 
FROM Track 
ORDER BY Bytes DESC;

-- Most recent alerts first
SELECT InvoiceId, CustomerId, InvoiceDate 
FROM Invoice 
ORDER BY InvoiceDate DESC;

-- Longest-duration events
SELECT Name, Milliseconds 
FROM Track 
ORDER BY Milliseconds DESC;
```

**SOC Scenario:** Prioritizing investigation queue

---

### 3.3 ORDER BY Multiple Columns
```sql
-- Sort by severity, then by size
SELECT Name, GenreId, Bytes 
FROM Track 
ORDER BY GenreId ASC, Bytes DESC;

-- Group by user, then by date
SELECT CustomerId, InvoiceDate, Total 
FROM Invoice 
ORDER BY CustomerId ASC, InvoiceDate DESC;
```

---

### 3.4 LIMIT (Top N results)
```sql
-- Top 10 largest transfers
SELECT Name, Bytes 
FROM Track 
ORDER BY Bytes DESC 
LIMIT 10;

-- Most recent 20 alerts
SELECT * FROM Invoice 
ORDER BY InvoiceDate DESC 
LIMIT 20;

-- Top 5 critical events
SELECT Name, GenreId 
FROM Track 
WHERE GenreId = 1 
ORDER BY Bytes DESC 
LIMIT 5;
```

**SOC Scenario:** Focusing on top threats for triage

---

### 3.5 OFFSET (Pagination)
```sql
-- Skip first 10, get next 10
SELECT * FROM Invoice 
ORDER BY InvoiceDate DESC 
LIMIT 10 OFFSET 10;

-- Page 3 of results (assuming 20 per page)
SELECT * FROM Track 
ORDER BY TrackId 
LIMIT 20 OFFSET 40;
```

**SOC Scenario:** Building paginated dashboards

---

### 3.6 Combined: Complex Triage Query
```sql
-- Top 10 recent critical/high events with large data transfers
SELECT 
    t.Name AS Event,
    t.Bytes,
    g.Name AS Severity,
    a.Title AS Campaign
FROM Track t
JOIN Genre g ON t.GenreId = g.GenreId
JOIN Album a ON t.AlbumId = a.AlbumId
WHERE t.GenreId IN (1, 2)  -- Critical or High
  AND t.Bytes > 1048576    -- > 1MB
ORDER BY t.Bytes DESC, t.GenreId ASC
LIMIT 10;
```

---

## 4. AGGREGATION FUNCTIONS

### 4.1 COUNT(*) - Total Rows
```sql
-- Total number of alerts
SELECT COUNT(*) AS TotalAlerts 
FROM Invoice;

-- Total events in database
SELECT COUNT(*) AS TotalEvents 
FROM Track;

-- How many users exist?
SELECT COUNT(*) AS TotalUsers 
FROM Customer;
```

**SOC Scenario:** Volume metrics for reporting

---

### 4.2 COUNT(column) - Non-NULL values
```sql
-- How many events have a composer field?
SELECT COUNT(Composer) AS EventsWithComposer 
FROM Track;

-- Alerts with billing address
SELECT COUNT(BillingAddress) AS AlertsWithBilling 
FROM Invoice;

-- Difference shows missing data
SELECT 
    COUNT(*) AS TotalTracks,
    COUNT(Composer) AS TracksWithComposer,
    COUNT(*) - COUNT(Composer) AS MissingComposer
FROM Track;
```

**SOC Scenario:** Data quality assessment

---

### 4.3 COUNT(DISTINCT column)
```sql
-- How many unique users triggered alerts?
SELECT COUNT(DISTINCT CustomerId) AS UniqueUsersWithAlerts 
FROM Invoice;

-- How many different campaigns are active?
SELECT COUNT(DISTINCT AlbumId) AS ActiveCampaigns 
FROM Track;

-- Unique event types seen
SELECT COUNT(DISTINCT MediaTypeId) AS UniqueEventTypes 
FROM Track;
```

**SOC Scenario:** Measuring attack surface

---

### 4.4 SUM() - Total of numeric column
```sql
-- Total bytes transferred across all events
SELECT SUM(Bytes) AS TotalBytesTransferred 
FROM Track;

-- Total alert severity scores
SELECT SUM(Total) AS TotalSeverityScore 
FROM Invoice;

-- Total duration of all events (in ms)
SELECT SUM(Milliseconds) AS TotalEventDuration 
FROM Track;
```

**SOC Scenario:** Measuring total impact

---

### 4.5 AVG() - Average value
```sql
-- Average bytes per event
SELECT AVG(Bytes) AS AvgBytesPerEvent 
FROM Track;

-- Average alert severity
SELECT AVG(Total) AS AvgAlertSeverity 
FROM Invoice;

-- Average event duration
SELECT AVG(Milliseconds) AS AvgEventDurationMs 
FROM Track;
```

**SOC Scenario:** Establishing baselines

---

### 4.6 MIN() and MAX()
```sql
-- Smallest and largest data transfers
SELECT 
    MIN(Bytes) AS SmallestTransfer,
    MAX(Bytes) AS LargestTransfer 
FROM Track;

-- First and last alert timestamps
SELECT 
    MIN(InvoiceDate) AS FirstAlert,
    MAX(InvoiceDate) AS LastAlert 
FROM Invoice;

-- Severity range
SELECT 
    MIN(GenreId) AS LowestSeverity,
    MAX(GenreId) AS HighestSeverity 
FROM Track;
```

**SOC Scenario:** Finding extremes and outliers

---

### 4.7 Multiple Aggregations Together
```sql
-- Complete statistics for events
SELECT 
    COUNT(*) AS TotalEvents,
    COUNT(DISTINCT AlbumId) AS UniqueCampaigns,
    SUM(Bytes) AS TotalBytes,
    AVG(Bytes) AS AvgBytes,
    MIN(Bytes) AS MinBytes,
    MAX(Bytes) AS MaxBytes,
    MIN(Milliseconds) AS ShortestEvent,
    MAX(Milliseconds) AS LongestEvent
FROM Track;
```

**SOC Scenario:** Comprehensive event analysis report

---

## 5. GROUP BY and HAVING

### 5.1 Simple GROUP BY
```sql
-- Count events per campaign
SELECT 
    AlbumId,
    COUNT(*) AS EventCount 
FROM Track 
GROUP BY AlbumId;

-- Count alerts per user
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY CustomerId;

-- Events by severity
SELECT 
    GenreId,
    COUNT(*) AS EventCount 
FROM Track 
GROUP BY GenreId;
```

**SOC Scenario:** Threat pattern identification

---

### 5.2 GROUP BY with Aggregations
```sql
-- Total bytes transferred per campaign
SELECT 
    AlbumId,
    SUM(Bytes) AS TotalBytesTransferred,
    COUNT(*) AS EventCount 
FROM Track 
GROUP BY AlbumId;

-- Alert statistics per user
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount,
    SUM(Total) AS TotalSeverity,
    AVG(Total) AS AvgSeverity,
    MAX(Total) AS MaxSeverity 
FROM Invoice 
GROUP BY CustomerId;
```

**SOC Scenario:** User behavior profiling

---

### 5.3 GROUP BY Multiple Columns
```sql
-- Events by campaign and severity
SELECT 
    AlbumId,
    GenreId,
    COUNT(*) AS EventCount 
FROM Track 
GROUP BY AlbumId, GenreId 
ORDER BY AlbumId, GenreId;

-- Events by campaign and event type
SELECT 
    AlbumId,
    MediaTypeId,
    COUNT(*) AS EventCount,
    SUM(Bytes) AS TotalBytes 
FROM Track 
GROUP BY AlbumId, MediaTypeId;
```

---

### 5.4 GROUP BY with JOINs (readable output)
```sql
-- Events per campaign with names
SELECT 
    a.Title AS Campaign,
    COUNT(*) AS EventCount,
    SUM(t.Bytes) AS TotalBytes 
FROM Track t
JOIN Album a ON t.AlbumId = a.AlbumId 
GROUP BY a.Title 
ORDER BY EventCount DESC;

-- Severity distribution with names
SELECT 
    g.Name AS Severity,
    COUNT(*) AS EventCount 
FROM Track t
JOIN Genre g ON t.GenreId = g.GenreId 
GROUP BY g.Name 
ORDER BY 
    CASE g.Name 
        WHEN 'Critical' THEN 1
        WHEN 'High' THEN 2
        WHEN 'Medium' THEN 3
        WHEN 'Low' THEN 4
        ELSE 5 
    END;
```

**SOC Scenario:** Executive reporting

---

### 5.5 HAVING - Filter Groups
```sql
-- Users with more than 10 alerts (high-risk users)
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY CustomerId 
HAVING COUNT(*) > 10;

-- Campaigns with significant data transfer (> 50MB)
SELECT 
    AlbumId,
    SUM(Bytes) AS TotalBytes 
FROM Track 
GROUP BY AlbumId 
HAVING SUM(Bytes) > 52428800;
```

**SOC Scenario:** Finding abnormal behavior patterns

---

### 5.6 HAVING with Multiple Conditions
```sql
-- High-activity, high-severity users
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount,
    AVG(Total) AS AvgSeverity 
FROM Invoice 
GROUP BY CustomerId 
HAVING COUNT(*) > 5 AND AVG(Total) > 3;

-- Campaigns with many high-severity events
SELECT 
    AlbumId,
    COUNT(*) AS EventCount 
FROM Track 
WHERE GenreId IN (1, 2)  -- WHERE filters rows first
GROUP BY AlbumId 
HAVING COUNT(*) > 10;    -- HAVING filters groups after
```

---

### 5.7 WHERE vs HAVING - Critical Difference
```sql
-- WHERE filters BEFORE grouping (filters individual rows)
-- HAVING filters AFTER grouping (filters aggregate results)

-- WRONG: Can't use aggregate in WHERE
-- SELECT CustomerId, COUNT(*) FROM Invoice WHERE COUNT(*) > 5;  -- ERROR!

-- CORRECT: Use HAVING for aggregates
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY CustomerId 
HAVING COUNT(*) > 5;

-- CORRECT: Combine WHERE and HAVING
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount 
FROM Invoice 
WHERE InvoiceDate >= '2025-01-01'  -- Filter rows first
GROUP BY CustomerId 
HAVING COUNT(*) > 5;               -- Filter groups after
```

**SOC Scenario:** Building detection rules

---

### 5.8 Complex GROUP BY Example
```sql
-- Comprehensive campaign analysis
SELECT 
    a.Title AS Campaign,
    ar.Name AS ThreatActor,
    COUNT(DISTINCT t.TrackId) AS UniqueEvents,
    COUNT(DISTINCT t.MediaTypeId) AS EventTypes,
    SUM(t.Bytes) AS TotalDataTransferred,
    AVG(t.Bytes) AS AvgDataPerEvent,
    MAX(t.Bytes) AS LargestTransfer,
    SUM(CASE WHEN t.GenreId = 1 THEN 1 ELSE 0 END) AS CriticalEvents,
    SUM(CASE WHEN t.GenreId = 2 THEN 1 ELSE 0 END) AS HighEvents 
FROM Track t
JOIN Album a ON t.AlbumId = a.AlbumId
JOIN Artist ar ON a.ArtistId = ar.ArtistId 
GROUP BY a.Title, ar.Name 
HAVING COUNT(DISTINCT t.TrackId) > 5 
ORDER BY CriticalEvents DESC, TotalDataTransferred DESC;
```

---

## 6. JOINS (Correlation Across Tables)

### 6.1 INNER JOIN (matching records only)
```sql
-- Events with their severity names
SELECT 
    t.TrackId,
    t.Name AS EventName,
    g.Name AS Severity 
FROM Track t
INNER JOIN Genre g ON t.GenreId = g.GenreId;

-- Events with campaign names
SELECT 
    t.Name AS Event,
    a.Title AS Campaign 
FROM Track t
INNER JOIN Album a ON t.AlbumId = a.AlbumId;

-- Alerts with user information
SELECT 
    i.InvoiceId,
    i.InvoiceDate,
    c.FirstName,
    c.LastName,
    c.Email 
FROM Invoice i
INNER JOIN Customer c ON i.CustomerId = c.CustomerId;
```

**SOC Scenario:** Enriching alerts with context

---

### 6.2 Multiple JOINs (correlation chain)
```sql
-- Events with all context: campaign, threat actor, severity
SELECT 
    t.Name AS Event,
    a.Title AS Campaign,
    ar.Name AS ThreatActor,
    g.Name AS Severity,
    m.Name AS EventType,
    t.Bytes 
FROM Track t
INNER JOIN Album a ON t.AlbumId = a.AlbumId
INNER JOIN Artist ar ON a.ArtistId = ar.ArtistId
INNER JOIN Genre g ON t.GenreId = g.GenreId
INNER JOIN MediaType m ON t.MediaTypeId = m.MediaTypeId 
WHERE t.GenreId <= 2  -- Critical or High
ORDER BY t.Bytes DESC 
LIMIT 20;
```

**SOC Scenario:** Complete event enrichment for investigation

---

### 6.3 LEFT JOIN (all from left table, matched from right)
```sql
-- All users, with or without alerts
SELECT 
    c.CustomerId,
    c.FirstName,
    c.LastName,
    c.Email,
    COUNT(i.InvoiceId) AS AlertCount 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId 
GROUP BY c.CustomerId, c.FirstName, c.LastName, c.Email 
ORDER BY AlertCount DESC;

-- Finding users with NO alerts (potentially missing telemetry)
SELECT 
    c.CustomerId,
    c.FirstName,
    c.LastName,
    c.Email 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId 
WHERE i.InvoiceId IS NULL;
```

**SOC Scenario:** Finding blind spots in coverage

---

### 6.4 RIGHT JOIN (less common)
```sql
-- Same as LEFT JOIN but reversed
-- All alerts, with user info if available
SELECT 
    i.InvoiceId,
    i.InvoiceDate,
    c.FirstName,
    c.LastName 
FROM Customer c
RIGHT JOIN Invoice i ON c.CustomerId = i.CustomerId;

-- Note: Usually written as LEFT JOIN instead:
SELECT 
    i.InvoiceId,
    i.InvoiceDate,
    c.FirstName,
    c.LastName 
FROM Invoice i
LEFT JOIN Customer c ON i.CustomerId = c.CustomerId;
```

---

### 6.5 FULL OUTER JOIN (all records from both)
```sql
-- All users and all alerts, matched where possible
-- Note: SQLite doesn't support FULL OUTER JOIN natively
-- Use UNION of LEFT JOINs instead:

SELECT 
    c.CustomerId,
    c.Email,
    i.InvoiceId,
    i.InvoiceDate 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId

UNION

SELECT 
    c.CustomerId,
    c.Email,
    i.InvoiceId,
    i.InvoiceDate 
FROM Invoice i
LEFT JOIN Customer c ON i.CustomerId = c.CustomerId 
WHERE c.CustomerId IS NULL;
```

---

### 6.6 Self JOIN (correlating table to itself)
```sql
-- Find SOC analysts and their managers
SELECT 
    e1.FirstName || ' ' || e1.LastName AS Analyst,
    e1.Title AS AnalystTitle,
    e2.FirstName || ' ' || e2.LastName AS Manager,
    e2.Title AS ManagerTitle 
FROM Employee e1
LEFT JOIN Employee e2 ON e1.ReportsTo = e2.EmployeeId;

-- Find users from same company
SELECT 
    c1.FirstName || ' ' || c1.LastName AS User1,
    c2.FirstName || ' ' || c2.LastName AS User2,
    c1.Company 
FROM Customer c1
JOIN Customer c2 ON c1.Company = c2.Company 
WHERE c1.CustomerId < c2.CustomerId  -- Avoid duplicates
ORDER BY c1.Company;
```

**SOC Scenario:** Finding related entities

---

### 6.7 JOIN with Multiple Conditions
```sql
-- Complex correlation: alerts and events
SELECT 
    i.InvoiceId,
    c.Email,
    t.Name AS EventName,
    g.Name AS Severity 
FROM Invoice i
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId
JOIN Genre g ON t.GenreId = g.GenreId AND g.GenreId <= 2  -- Only Critical/High
JOIN Customer c ON i.CustomerId = c.CustomerId 
WHERE i.InvoiceDate >= '2025-01-20' 
ORDER BY i.InvoiceDate DESC;
```

---

### 6.8 Complex Multi-Table Correlation
```sql
-- Complete alert investigation view
SELECT 
    i.InvoiceId AS AlertID,
    i.InvoiceDate AS Timestamp,
    c.FirstName || ' ' || c.LastName AS User,
    c.Email,
    c.Company,
    e.FirstName || ' ' || e.LastName AS AssignedAnalyst,
    t.Name AS EventName,
    m.Name AS EventType,
    g.Name AS Severity,
    a.Title AS Campaign,
    ar.Name AS ThreatActor,
    t.Bytes AS DataTransferred 
FROM Invoice i
JOIN Customer c ON i.CustomerId = c.CustomerId
LEFT JOIN Employee e ON c.SupportRepId = e.EmployeeId
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId
JOIN MediaType m ON t.MediaTypeId = m.MediaTypeId
JOIN Genre g ON t.GenreId = g.GenreId
LEFT JOIN Album a ON t.AlbumId = a.AlbumId
LEFT JOIN Artist ar ON a.ArtistId = ar.ArtistId 
WHERE g.GenreId <= 3  -- Critical, High, Medium
ORDER BY g.GenreId ASC, t.Bytes DESC 
LIMIT 50;
```

**SOC Scenario:** Analyst workbench query

---

## 7. SUBQUERIES (Pivoting Investigations)

### 7.1 Subquery in WHERE Clause (filtering)
```sql
-- Events from campaigns by specific threat actor
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

-- Users with above-average alert counts
SELECT 
    CustomerId,
    FirstName,
    LastName,
    Email 
FROM Customer 
WHERE CustomerId IN (
    SELECT CustomerId 
    FROM Invoice 
    GROUP BY CustomerId 
    HAVING COUNT(*) > (
        SELECT AVG(alert_count) 
        FROM (
            SELECT COUNT(*) AS alert_count 
            FROM Invoice 
            GROUP BY CustomerId
        )
    )
);
```

**SOC Scenario:** Finding outliers

---

### 7.2 Scalar Subquery (returns single value)
```sql
-- Compare each event size to average
SELECT 
    Name,
    Bytes,
    (SELECT AVG(Bytes) FROM Track) AS AvgBytes,
    Bytes - (SELECT AVG(Bytes) FROM Track) AS DifferenceFromAvg 
FROM Track 
WHERE Bytes > (SELECT AVG(Bytes) FROM Track)
ORDER BY Bytes DESC;

-- Alerts above average severity
SELECT 
    InvoiceId,
    CustomerId,
    Total,
    (SELECT AVG(Total) FROM Invoice) AS AvgSeverity 
FROM Invoice 
WHERE Total > (SELECT AVG(Total) FROM Invoice);
```

**SOC Scenario:** Baseline deviation detection

---

### 7.3 List Subquery (IN operator)
```sql
-- Events from high-severity campaigns
SELECT 
    Name,
    AlbumId,
    GenreId 
FROM Track 
WHERE AlbumId IN (
    SELECT DISTINCT AlbumId 
    FROM Track 
    WHERE GenreId <= 2  -- Critical or High
    GROUP BY AlbumId 
    HAVING COUNT(*) > 5
);

-- Users who triggered critical alerts
SELECT * 
FROM Customer 
WHERE CustomerId IN (
    SELECT DISTINCT i.CustomerId 
    FROM Invoice i
    JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
    JOIN Track t ON il.TrackId = t.TrackId 
    WHERE t.GenreId = 1  -- Critical
);
```

---

### 7.4 Subquery in FROM Clause (derived table)
```sql
-- Alert statistics as a derived table
SELECT 
    user_stats.CustomerId,
    c.Email,
    user_stats.AlertCount,
    user_stats.TotalSeverity 
FROM (
    SELECT 
        CustomerId,
        COUNT(*) AS AlertCount,
        SUM(Total) AS TotalSeverity 
    FROM Invoice 
    GROUP BY CustomerId
) AS user_stats
JOIN Customer c ON user_stats.CustomerId = c.CustomerId 
WHERE user_stats.AlertCount > 10 
ORDER BY user_stats.AlertCount DESC;

-- Top campaigns by event volume
SELECT 
    campaign_stats.*,
    a.Title AS CampaignName,
    ar.Name AS ThreatActor 
FROM (
    SELECT 
        AlbumId,
        COUNT(*) AS EventCount,
        SUM(Bytes) AS TotalBytes,
        AVG(Bytes) AS AvgBytes 
    FROM Track 
    GROUP BY AlbumId 
    HAVING COUNT(*) > 5
) AS campaign_stats
JOIN Album a ON campaign_stats.AlbumId = a.AlbumId
JOIN Artist ar ON a.ArtistId = ar.ArtistId 
ORDER BY campaign_stats.EventCount DESC;
```

**SOC Scenario:** Complex analytics

---

### 7.5 Correlated Subquery (references outer query)
```sql
-- For each user, count their alerts
SELECT 
    c.CustomerId,
    c.FirstName,
    c.LastName,
    (
        SELECT COUNT(*) 
        FROM Invoice i 
        WHERE i.CustomerId = c.CustomerId
    ) AS AlertCount 
FROM Customer c 
ORDER BY AlertCount DESC;

-- Events with above-average bytes for their severity
SELECT 
    t1.Name,
    t1.GenreId,
    t1.Bytes 
FROM Track t1 
WHERE t1.Bytes > (
    SELECT AVG(t2.Bytes) 
    FROM Track t2 
    WHERE t2.GenreId = t1.GenreId  -- Correlated: same severity
);
```

---

### 7.6 Subquery in SELECT Clause
```sql
-- User details with alert count
SELECT 
    CustomerId,
    FirstName,
    LastName,
    Email,
    (SELECT COUNT(*) FROM Invoice i WHERE i.CustomerId = Customer.CustomerId) AS AlertCount,
    (SELECT MAX(InvoiceDate) FROM Invoice i WHERE i.CustomerId = Customer.CustomerId) AS LastAlert 
FROM Customer 
ORDER BY AlertCount DESC;

-- Campaign with threat actor name
SELECT 
    AlbumId,
    Title,
    (SELECT Name FROM Artist ar WHERE ar.ArtistId = Album.ArtistId) AS ThreatActor 
FROM Album;
```

---

### 7.7 Nested Subqueries (subquery within subquery)
```sql
-- Users from companies with the most alerts
SELECT * 
FROM Customer 
WHERE Company IN (
    SELECT Company 
    FROM (
        SELECT 
            c.Company,
            COUNT(DISTINCT i.InvoiceId) AS AlertCount 
        FROM Customer c
        JOIN Invoice i ON c.CustomerId = i.CustomerId 
        WHERE c.Company IS NOT NULL 
        GROUP BY c.Company 
        ORDER BY AlertCount DESC 
        LIMIT 3
    )
);

-- Events from top threat actors' top campaigns
SELECT * 
FROM Track 
WHERE AlbumId IN (
    SELECT AlbumId 
    FROM Album 
    WHERE ArtistId IN (
        SELECT ArtistId 
        FROM (
            SELECT 
                a.ArtistId,
                COUNT(t.TrackId) AS EventCount 
            FROM Artist a
            JOIN Album al ON a.ArtistId = al.ArtistId
            JOIN Track t ON al.AlbumId = t.AlbumId 
            GROUP BY a.ArtistId 
            ORDER BY EventCount DESC 
            LIMIT 5
        )
    )
);
```

**SOC Scenario:** Deep dive investigations

---

## 8. CASE EXPRESSIONS (Classification & Labeling)

### 8.1 Simple CASE - Event Severity Tagging
```sql
-- Convert numeric severity to labels
SELECT 
    Name,
    GenreId,
    CASE GenreId
        WHEN 1 THEN 'CRITICAL'
        WHEN 2 THEN 'HIGH'
        WHEN 3 THEN 'MEDIUM'
        WHEN 4 THEN 'LOW'
        WHEN 5 THEN 'INFO'
        ELSE 'UNKNOWN'
    END AS SeverityLabel 
FROM Track;
```

---

### 8.2 Searched CASE - Complex Conditions
```sql
-- Classify data transfer size
SELECT 
    Name,
    Bytes,
    CASE 
        WHEN Bytes < 1024 THEN 'Tiny'
        WHEN Bytes < 1048576 THEN 'Small'
        WHEN Bytes < 10485760 THEN 'Medium'
        WHEN Bytes < 104857600 THEN 'Large'
        ELSE 'Very Large - Suspicious'
    END AS TransferSize 
FROM Track 
ORDER BY Bytes DESC;

-- Risk scoring based on multiple factors
SELECT 
    t.Name,
    t.GenreId,
    t.Bytes,
    CASE 
        WHEN t.GenreId = 1 AND t.Bytes > 10485760 THEN 'CRITICAL - Large Data + Critical Event'
        WHEN t.GenreId = 1 THEN 'CRITICAL - Requires Immediate Action'
        WHEN t.GenreId = 2 AND t.Bytes > 10485760 THEN 'HIGH - Large Data Transfer'
        WHEN t.GenreId = 2 THEN 'HIGH - Review Required'
        WHEN t.Bytes > 52428800 THEN 'MEDIUM - Unusual Data Volume'
        ELSE 'LOW - Normal Activity'
    END AS RiskAssessment 
FROM Track 
ORDER BY GenreId, Bytes DESC;
```

**SOC Scenario:** Automated alert prioritization

---

### 8.3 CASE in Aggregations
```sql
-- Count events by custom categories
SELECT 
    SUM(CASE WHEN GenreId = 1 THEN 1 ELSE 0 END) AS CriticalCount,
    SUM(CASE WHEN GenreId = 2 THEN 1 ELSE 0 END) AS HighCount,
    SUM(CASE WHEN GenreId = 3 THEN 1 ELSE 0 END) AS MediumCount,
    SUM(CASE WHEN GenreId = 4 THEN 1 ELSE 0 END) AS LowCount,
    SUM(CASE WHEN GenreId = 5 THEN 1 ELSE 0 END) AS InfoCount 
FROM Track;

-- Calculate percentage by severity
SELECT 
    COUNT(*) AS TotalEvents,
    ROUND(SUM(CASE WHEN GenreId <= 2 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS PctHighPriority,
    ROUND(SUM(CASE WHEN GenreId = 3 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS PctMedium,
    ROUND(SUM(CASE WHEN GenreId >= 4 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS PctLowPriority 
FROM Track;
```

---

### 8.4 CASE with GROUP BY
```sql
-- Alert volume by time period
SELECT 
    CASE 
        WHEN InvoiceDate >= '2025-01-25' THEN 'Last 7 Days'
        WHEN InvoiceDate >= '2025-01-18' THEN 'Last 14 Days'
        WHEN InvoiceDate >= '2025-01-04' THEN 'Last 30 Days'
        ELSE 'Older'
    END AS TimePeriod,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY 
    CASE 
        WHEN InvoiceDate >= '2025-01-25' THEN 'Last 7 Days'
        WHEN InvoiceDate >= '2025-01-18' THEN 'Last 14 Days'
        WHEN InvoiceDate >= '2025-01-04' THEN 'Last 30 Days'
        ELSE 'Older'
    END 
ORDER BY MIN(InvoiceDate) DESC;
```

---

### 8.5 Business Hours Detection
```sql
-- Classify activity by time (assuming we add hour to data)
SELECT 
    Name,
    CAST(strftime('%H', InvoiceDate) AS INTEGER) AS Hour,
    CASE 
        WHEN CAST(strftime('%H', InvoiceDate) AS INTEGER) BETWEEN 9 AND 17 THEN 'Business Hours'
        WHEN CAST(strftime('%H', InvoiceDate) AS INTEGER) BETWEEN 18 AND 23 THEN 'After Hours - Evening'
        ELSE 'After Hours - Night/Early Morning'
    END AS TimeClassification 
FROM Invoice i
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId;
```

**SOC Scenario:** After-hours activity detection

---

### 8.6 Multi-Factor Risk Scoring
```sql
-- Complex risk calculation
SELECT 
    c.Email,
    COUNT(i.InvoiceId) AS AlertCount,
    AVG(i.Total) AS AvgSeverity,
    CASE 
        WHEN COUNT(i.InvoiceId) > 20 AND AVG(i.Total) > 5 THEN 'CRITICAL RISK'
        WHEN COUNT(i.InvoiceId) > 15 OR AVG(i.Total) > 4 THEN 'HIGH RISK'
        WHEN COUNT(i.InvoiceId) > 10 THEN 'MEDIUM RISK'
        WHEN COUNT(i.InvoiceId) > 5 THEN 'LOW RISK'
        ELSE 'MINIMAL RISK'
    END AS RiskLevel,
    CASE 
        WHEN COUNT(i.InvoiceId) > 20 THEN 'Immediate investigation required'
        WHEN COUNT(i.InvoiceId) > 10 THEN 'Review within 24 hours'
        WHEN COUNT(i.InvoiceId) > 5 THEN 'Monitor closely'
        ELSE 'Standard monitoring'
    END AS Recommendation 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId 
GROUP BY c.CustomerId, c.Email 
HAVING COUNT(i.InvoiceId) > 0 
ORDER BY AlertCount DESC;
```

---

## 9. STRING FUNCTIONS (Logs, Malware, Commands)

### 9.1 LOWER() and UPPER()
```sql
-- Case-insensitive search
SELECT * 
FROM Track 
WHERE LOWER(Name) LIKE '%login%';

-- Standardize output
SELECT 
    UPPER(Name) AS EventName,
    LOWER(Composer) AS NormalizedComposer 
FROM Track 
WHERE Composer IS NOT NULL;

-- Case-insensitive comparison
SELECT * 
FROM Customer 
WHERE LOWER(Email) LIKE '%techcorp%';
```

**SOC Scenario:** Parsing inconsistent log formats

---

### 9.2 LENGTH() - String Length
```sql
-- Find unusually long event names (possible obfuscation)
SELECT 
    Name,
    LENGTH(Name) AS NameLength 
FROM Track 
WHERE LENGTH(Name) > 30 
ORDER BY NameLength DESC;

-- Email validation
SELECT 
    Email,
    LENGTH(Email) AS EmailLength 
FROM Customer 
WHERE LENGTH(Email) < 5 OR LENGTH(Email) > 50;

-- Command length analysis
SELECT 
    Name,
    LENGTH(Name) AS CommandLength,
    CASE 
        WHEN LENGTH(Name) > 100 THEN 'Suspicious - Very Long'
        WHEN LENGTH(Name) > 50 THEN 'Review - Long Command'
        ELSE 'Normal'
    END AS LengthAssessment 
FROM Track 
WHERE MediaTypeId = 9;  -- PowerShell commands
```

---

### 9.3 SUBSTRING() / SUBSTR()
```sql
-- Extract domain from email
SELECT 
    Email,
    SUBSTR(Email, INSTR(Email, '@') + 1) AS Domain 
FROM Customer;

-- Extract first word from event name
SELECT 
    Name,
    SUBSTR(Name, 1, INSTR(Name || ' ', ' ') - 1) AS FirstWord 
FROM Track;

-- Extract year from date
SELECT 
    InvoiceDate,
    SUBSTR(InvoiceDate, 1, 4) AS Year,
    SUBSTR(InvoiceDate, 6, 2) AS Month 
FROM Invoice;
```

**SOC Scenario:** Parsing structured logs

---

### 9.4 TRIM(), LTRIM(), RTRIM()
```sql
-- Remove whitespace
SELECT 
    '  ' || Name || '  ' AS Original,
    TRIM('  ' || Name || '  ') AS Trimmed 
FROM Track 
LIMIT 5;

-- Clean user input
SELECT 
    TRIM(FirstName) AS CleanFirstName,
    TRIM(LastName) AS CleanLastName 
FROM Customer;
```

---

### 9.5 REPLACE()
```sql
-- Normalize event names
SELECT 
    Name AS Original,
    REPLACE(Name, 'Login', 'Authentication') AS Normalized 
FROM Track 
WHERE Name LIKE '%Login%';

-- Remove special characters
SELECT 
    Name,
    REPLACE(REPLACE(Name, '-', ' '), '_', ' ') AS Cleaned 
FROM Track;

-- Redact sensitive data
SELECT 
    Email,
    REPLACE(Email, SUBSTR(Email, 1, INSTR(Email, '@') - 1), '***') AS RedactedEmail 
FROM Customer;
```

**SOC Scenario:** Log normalization

---

### 9.6 CONCAT() or ||
```sql
-- Combine user name
SELECT 
    FirstName || ' ' || LastName AS FullName,
    Email 
FROM Customer;

-- Build descriptive event label
SELECT 
    Name || ' (' || GenreId || ')' AS EventWithSeverity 
FROM Track;

-- Create alert description
SELECT 
    'Alert #' || InvoiceId || ' for user ' || CustomerId || ' at ' || InvoiceDate AS AlertDescription 
FROM Invoice 
LIMIT 10;
```

---

### 9.7 INSTR() - Find Substring Position
```sql
-- Find position of @ in email
SELECT 
    Email,
    INSTR(Email, '@') AS AtPosition 
FROM Customer;

-- Check if event contains keyword
SELECT 
    Name,
    INSTR(LOWER(Name), 'failed') AS ContainsFailed 
FROM Track 
WHERE INSTR(LOWER(Name), 'failed') > 0;
```

---

### 9.8 Complex String Parsing
```sql
-- Parse email into username and domain
SELECT 
    Email,
    SUBSTR(Email, 1, INSTR(Email, '@') - 1) AS Username,
    SUBSTR(Email, INSTR(Email, '@') + 1) AS Domain,
    CASE 
        WHEN SUBSTR(Email, INSTR(Email, '@') + 1) LIKE '%.com' THEN 'Commercial'
        WHEN SUBSTR(Email, INSTR(Email, '@') + 1) LIKE '%.gov' THEN 'Government'
        WHEN SUBSTR(Email, INSTR(Email, '@') + 1) LIKE '%.edu' THEN 'Educational'
        ELSE 'Other'
    END AS DomainType 
FROM Customer;

-- Extract and classify event type from name
SELECT 
    Name,
    SUBSTR(Name, 1, INSTR(Name || ' ', ' ') - 1) AS Action,
    CASE 
        WHEN LOWER(Name) LIKE '%failed%' THEN 'Failure'
        WHEN LOWER(Name) LIKE '%successful%' OR LOWER(Name) LIKE '%success%' THEN 'Success'
        WHEN LOWER(Name) LIKE '%attempt%' THEN 'Attempt'
        ELSE 'Other'
    END AS EventCategory 
FROM Track;
```

**SOC Scenario:** Advanced log parsing and enrichment

---

## 10. DATE & TIME FUNCTIONS (SOC Critical)

### 10.1 Current Time Functions
```sql
-- Current timestamp
SELECT 
    datetime('now') AS CurrentUTC,
    datetime('now', 'localtime') AS CurrentLocal;

-- Current date only
SELECT date('now') AS Today;

-- Current time components
SELECT 
    strftime('%Y', 'now') AS Year,
    strftime('%m', 'now') AS Month,
    strftime('%d', 'now') AS Day,
    strftime('%H', 'now') AS Hour,
    strftime('%M', 'now') AS Minute,
    strftime('%S', 'now') AS Second;
```

---

### 10.2 Date Extraction
```sql
-- Extract date components from alerts
SELECT 
    InvoiceDate,
    date(InvoiceDate) AS DateOnly,
    strftime('%Y', InvoiceDate) AS Year,
    strftime('%m', InvoiceDate) AS Month,
    strftime('%d', InvoiceDate) AS Day,
    strftime('%H', InvoiceDate) AS Hour,
    strftime('%w', InvoiceDate) AS DayOfWeek  -- 0=Sunday
FROM Invoice;

-- Get day name
SELECT 
    InvoiceDate,
    CASE CAST(strftime('%w', InvoiceDate) AS INTEGER)
        WHEN 0 THEN 'Sunday'
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
    END AS DayName 
FROM Invoice;
```

---

### 10.3 Date Filtering
```sql
-- Alerts from today
SELECT * 
FROM Invoice 
WHERE date(InvoiceDate) = date('now');

-- Alerts from specific month
SELECT * 
FROM Invoice 
WHERE strftime('%Y-%m', InvoiceDate) = '2025-01';

-- Alerts from last 7 days
SELECT * 
FROM Invoice 
WHERE InvoiceDate >= datetime('now', '-7 days');

-- Alerts from last 24 hours
SELECT * 
FROM Invoice 
WHERE InvoiceDate >= datetime('now', '-1 day');
```

**SOC Scenario:** Time-based alert filtering

---

### 10.4 Date Math (Adding/Subtracting)
```sql
-- Dates relative to now
SELECT 
    datetime('now') AS Now,
    datetime('now', '-1 hour') AS OneHourAgo,
    datetime('now', '-1 day') AS OneDayAgo,
    datetime('now', '-7 days') AS SevenDaysAgo,
    datetime('now', '-1 month') AS OneMonthAgo,
    datetime('now', '+1 day') AS Tomorrow;

-- Calculate alert age
SELECT 
    InvoiceId,
    InvoiceDate,
    CAST((julianday('now') - julianday(InvoiceDate)) AS INTEGER) AS DaysAgo,
    CAST((julianday('now') - julianday(InvoiceDate)) * 24 AS INTEGER) AS HoursAgo 
FROM Invoice 
ORDER BY InvoiceDate DESC;
```

---

### 10.5 Time Differences
```sql
-- Calculate time between alerts for same user
SELECT 
    i1.CustomerId,
    i1.InvoiceDate AS FirstAlert,
    i2.InvoiceDate AS SecondAlert,
    CAST((julianday(i2.InvoiceDate) - julianday(i1.InvoiceDate)) * 24 * 60 AS INTEGER) AS MinutesBetween 
FROM Invoice i1
JOIN Invoice i2 ON i1.CustomerId = i2.CustomerId 
    AND i1.InvoiceId < i2.InvoiceId 
WHERE i1.CustomerId = 1 
ORDER BY i1.InvoiceDate;

-- Average time between alerts per user
SELECT 
    CustomerId,
    COUNT(*) AS AlertCount,
    CAST((julianday(MAX(InvoiceDate)) - julianday(MIN(InvoiceDate))) AS REAL) AS DaysSpan,
    CASE 
        WHEN COUNT(*) > 1 THEN 
            ROUND((julianday(MAX(InvoiceDate)) - julianday(MIN(InvoiceDate))) / (COUNT(*) - 1), 2)
        ELSE NULL 
    END AS AvgDaysBetweenAlerts 
FROM Invoice 
GROUP BY CustomerId 
HAVING COUNT(*) > 1;
```

**SOC Scenario:** Burst detection

---

### 10.6 Business Hours / After-Hours Detection
```sql
-- Classify by business hours
SELECT 
    InvoiceId,
    InvoiceDate,
    CAST(strftime('%H', InvoiceDate) AS INTEGER) AS Hour,
    CASE 
        WHEN CAST(strftime('%H', InvoiceDate) AS INTEGER) BETWEEN 8 AND 17 THEN 'Business Hours'
        WHEN CAST(strftime('%H', InvoiceDate) AS INTEGER) BETWEEN 18 AND 22 THEN 'After Hours - Evening'
        ELSE 'After Hours - Night'
    END AS TimeClassification,
    CASE CAST(strftime('%w', InvoiceDate) AS INTEGER)
        WHEN 0 THEN 'Weekend'
        WHEN 6 THEN 'Weekend'
        ELSE 'Weekday'
    END AS DayType 
FROM Invoice;

-- Count after-hours activity
SELECT 
    CustomerId,
    COUNT(*) AS TotalAlerts,
    SUM(CASE 
        WHEN CAST(strftime('%H', InvoiceDate) AS INTEGER) NOT BETWEEN 8 AND 17 THEN 1 
        ELSE 0 
    END) AS AfterHoursAlerts,
    SUM(CASE 
        WHEN CAST(strftime('%w', InvoiceDate) AS INTEGER) IN (0, 6) THEN 1 
        ELSE 0 
    END) AS WeekendAlerts 
FROM Invoice 
GROUP BY CustomerId 
HAVING AfterHoursAlerts > 5 OR WeekendAlerts > 3;
```

**SOC Scenario:** Detecting suspicious timing

---

### 10.7 Time-Window Analysis
```sql
-- Alerts per hour of day
SELECT 
    strftime('%H', InvoiceDate) AS Hour,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY strftime('%H', InvoiceDate) 
ORDER BY Hour;

-- Alerts per day of week
SELECT 
    CASE CAST(strftime('%w', InvoiceDate) AS INTEGER)
        WHEN 0 THEN '0-Sunday'
        WHEN 1 THEN '1-Monday'
        WHEN 2 THEN '2-Tuesday'
        WHEN 3 THEN '3-Wednesday'
        WHEN 4 THEN '4-Thursday'
        WHEN 5 THEN '5-Friday'
        WHEN 6 THEN '6-Saturday'
    END AS DayOfWeek,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY CAST(strftime('%w', InvoiceDate) AS INTEGER) 
ORDER BY CAST(strftime('%w', InvoiceDate) AS INTEGER);

-- Daily alert trend
SELECT 
    date(InvoiceDate) AS Date,
    COUNT(*) AS AlertCount 
FROM Invoice 
GROUP BY date(InvoiceDate) 
ORDER BY Date DESC 
LIMIT 30;
```

**SOC Scenario:** Pattern identification

---

### 10.8 Time-Based Correlation
```sql
-- Find rapid-fire alerts (< 5 minutes apart)
SELECT 
    i1.InvoiceId AS Alert1,
    i2.InvoiceId AS Alert2,
    i1.CustomerId,
    i1.InvoiceDate AS Time1,
    i2.InvoiceDate AS Time2,
    CAST((julianday(i2.InvoiceDate) - julianday(i1.InvoiceDate)) * 24 * 60 AS INTEGER) AS MinutesApart 
FROM Invoice i1
JOIN Invoice i2 ON i1.CustomerId = i2.CustomerId 
    AND i1.InvoiceId < i2.InvoiceId 
    AND julianday(i2.InvoiceDate) - julianday(i1.InvoiceDate) < (5.0 / (24 * 60))
ORDER BY i1.CustomerId, i1.InvoiceDate;
```

**SOC Scenario:** Automated attack detection

---

## 11. MATHEMATICAL & LOGICAL EXPRESSIONS

### 11.1 Arithmetic Operators
```sql
-- Calculate MB from bytes
SELECT 
    Name,
    Bytes,
    Bytes / 1048576.0 AS MegaBytes,
    ROUND(Bytes / 1048576.0, 2) AS MB_Rounded 
FROM Track 
ORDER BY Bytes DESC 
LIMIT 20;

-- Calculate percentage
SELECT 
    GenreId,
    COUNT(*) AS EventCount,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM Track), 2) AS Percentage 
FROM Track 
GROUP BY GenreId;
```

---

### 11.2 Mathematical Functions
```sql
-- Rounding
SELECT 
    AVG(Bytes) AS AvgRaw,
    ROUND(AVG(Bytes)) AS AvgRounded,
    ROUND(AVG(Bytes), 2) AS AvgTwoDecimals 
FROM Track;

-- Absolute value
SELECT 
    Bytes,
    AVG(Bytes) OVER () AS AvgBytes,
    ABS(Bytes - AVG(Bytes) OVER ()) AS DeviationFromAvg 
FROM Track 
LIMIT 20;
```

---

### 11.3 Modulo (%) - Pattern Detection
```sql
-- Find events on even-numbered tracks (possible pattern)
SELECT * 
FROM Track 
WHERE TrackId % 2 = 0;

-- Group events by size buckets
SELECT 
    (Bytes / 1048576) AS MBucket,
    COUNT(*) AS EventCount 
FROM Track 
GROUP BY (Bytes / 1048576) 
ORDER BY MBucket;
```

---

### 11.4 CAST() and Type Conversion
```sql
-- Convert to integer
SELECT 
    Total,
    CAST(Total AS INTEGER) AS TotalInt,
    ROUND(Total) AS TotalRounded 
FROM Invoice;

-- Extract hour as integer
SELECT 
    InvoiceDate,
    strftime('%H', InvoiceDate) AS HourString,
    CAST(strftime('%H', InvoiceDate) AS INTEGER) AS HourInt 
FROM Invoice;
```

---

### 11.5 Ratio Calculations
```sql
-- Critical to total event ratio by campaign
SELECT 
    AlbumId,
    COUNT(*) AS TotalEvents,
    SUM(CASE WHEN GenreId = 1 THEN 1 ELSE 0 END) AS CriticalEvents,
    ROUND(
        SUM(CASE WHEN GenreId = 1 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 
        2
    ) AS CriticalPercentage 
FROM Track 
GROUP BY AlbumId 
HAVING COUNT(*) > 5 
ORDER BY CriticalPercentage DESC;

-- Alert-to-user ratio
SELECT 
    c.CustomerId,
    c.Email,
    COUNT(i.InvoiceId) AS Alerts,
    ROUND(COUNT(i.InvoiceId) * 1.0 / 
        (SELECT COUNT(*) FROM Invoice), 4) AS PortionOfAllAlerts 
FROM Customer c
JOIN Invoice i ON c.CustomerId = i.CustomerId 
GROUP BY c.CustomerId, c.Email 
ORDER BY Alerts DESC;
```

**SOC Scenario:** Risk scoring and prioritization

---

### 11.6 Threshold Logic
```sql
-- Flag high-risk events
SELECT 
    Name,
    Bytes,
    GenreId,
    (Bytes > 10485760) AS IsLargeTransfer,
    (GenreId <= 2) AS IsHighSeverity,
    (Bytes > 10485760 AND GenreId <= 2) AS IsCriticalThreat 
FROM Track 
WHERE (Bytes > 10485760 AND GenreId <= 2) = 1;
```

---

### 11.7 Statistical Calculations
```sql
-- Event size statistics
SELECT 
    COUNT(*) AS TotalEvents,
    AVG(Bytes) AS MeanBytes,
    MIN(Bytes) AS MinBytes,
    MAX(Bytes) AS MaxBytes,
    MAX(Bytes) - MIN(Bytes) AS Range,
    -- Approximate standard deviation
    ROUND(AVG(Bytes * Bytes) - AVG(Bytes) * AVG(Bytes), 2) AS VarianceApprox 
FROM Track;
```

---

## 12. UNION OPERATIONS (Multi-Source Visibility)

### 12.1 UNION (Remove Duplicates)
```sql
-- Combine users from different sources
SELECT Email, 'Customer' AS Source 
FROM Customer

UNION

SELECT Email, 'Employee' AS Source 
FROM Employee;

-- All unique event names and types
SELECT Name AS Item, 'Event' AS Type FROM Track
UNION
SELECT Name, 'Severity' FROM Genre
UNION
SELECT Name, 'EventType' FROM MediaType;
```

---

### 12.2 UNION ALL (Keep Duplicates - Faster)
```sql
-- All activity from multiple tables (keep dupes for counting)
SELECT InvoiceId AS ID, InvoiceDate AS Timestamp, 'Alert' AS ActivityType 
FROM Invoice

UNION ALL

SELECT TrackId, NULL, 'Event' 
FROM Track;

-- Combine critical events from different filters
SELECT TrackId, Name, 'HighSeverity' AS Reason 
FROM Track 
WHERE GenreId <= 2

UNION ALL

SELECT TrackId, Name, 'LargeTransfer' 
FROM Track 
WHERE Bytes > 10485760;
```

**SOC Scenario:** Creating unified event stream

---

### 12.3 UNION with ORDER BY
```sql
-- Recent activity across tables
SELECT 
    InvoiceId AS ID,
    InvoiceDate AS Timestamp,
    'Alert Created' AS EventType,
    CustomerId AS RelatedEntity 
FROM Invoice

UNION ALL

SELECT 
    TrackId,
    NULL,
    'Event Logged',
    AlbumId 
FROM Track

ORDER BY Timestamp DESC 
LIMIT 50;
```

---

### 12.4 Complex UNION Example
```sql
-- Comprehensive threat summary
SELECT 
    ar.Name AS ThreatActor,
    a.Title AS Campaign,
    COUNT(t.TrackId) AS EventCount,
    'Active Campaign' AS Status 
FROM Artist ar
JOIN Album a ON ar.ArtistId = a.ArtistId
JOIN Track t ON a.AlbumId = t.AlbumId 
GROUP BY ar.Name, a.Title

UNION

SELECT 
    Name,
    'N/A',
    0,
    'Known Actor - No Recent Activity' 
FROM Artist 
WHERE ArtistId NOT IN (
    SELECT DISTINCT ArtistId FROM Album
)

ORDER BY EventCount DESC;
```

---

## 13. EXISTS / NOT EXISTS

### 13.1 EXISTS (Check for Related Records)
```sql
-- Users who have triggered alerts
SELECT 
    CustomerId,
    FirstName,
    LastName,
    Email 
FROM Customer c 
WHERE EXISTS (
    SELECT 1 
    FROM Invoice i 
    WHERE i.CustomerId = c.CustomerId
);

-- Threat actors with active campaigns
SELECT * 
FROM Artist ar 
WHERE EXISTS (
    SELECT 1 
    FROM Album a 
    WHERE a.ArtistId = ar.ArtistId
);

-- Campaigns with critical events
SELECT 
    AlbumId,
    Title 
FROM Album a 
WHERE EXISTS (
    SELECT 1 
    FROM Track t 
    WHERE t.AlbumId = a.AlbumId 
        AND t.GenreId = 1  -- Critical
);
```

**SOC Scenario:** Finding active threats

---

### 13.2 NOT EXISTS (Find Missing Relationships)
```sql
-- Users with NO alerts (blind spots)
SELECT 
    CustomerId,
    FirstName,
    LastName,
    Email,
    Company 
FROM Customer c 
WHERE NOT EXISTS (
    SELECT 1 
    FROM Invoice i 
    WHERE i.CustomerId = c.CustomerId
);

-- Threat actors with no campaigns (dormant threats)
SELECT * 
FROM Artist ar 
WHERE NOT EXISTS (
    SELECT 1 
    FROM Album a 
    WHERE a.ArtistId = ar.ArtistId
);

-- Events not in any watchlist
SELECT 
    TrackId,
    Name 
FROM Track t 
WHERE NOT EXISTS (
    SELECT 1 
    FROM PlaylistTrack pt 
    WHERE pt.TrackId = t.TrackId
);
```

**SOC Scenario:** Finding coverage gaps

---

### 13.3 EXISTS vs IN (Performance)
```sql
-- Both queries do same thing, but EXISTS is often faster
-- Using IN:
SELECT * FROM Customer 
WHERE CustomerId IN (
    SELECT CustomerId FROM Invoice
);

-- Using EXISTS (generally faster):
SELECT * FROM Customer c 
WHERE EXISTS (
    SELECT 1 FROM Invoice i 
    WHERE i.CustomerId = c.CustomerId
);
```

---

### 13.4 Complex NOT EXISTS (Gap Analysis)
```sql
-- Users from TechCorp with no critical alerts
SELECT 
    c.CustomerId,
    c.Email,
    c.Company 
FROM Customer c 
WHERE c.Company = 'TechCorp'
    AND NOT EXISTS (
        SELECT 1 
        FROM Invoice i
        JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
        JOIN Track t ON il.TrackId = t.TrackId 
        WHERE i.CustomerId = c.CustomerId 
            AND t.GenreId = 1  -- Critical
    );

-- Campaigns without high-severity events
SELECT 
    a.AlbumId,
    a.Title,
    ar.Name AS ThreatActor 
FROM Album a
JOIN Artist ar ON a.ArtistId = ar.ArtistId 
WHERE NOT EXISTS (
    SELECT 1 
    FROM Track t 
    WHERE t.AlbumId = a.AlbumId 
        AND t.GenreId <= 2  -- Critical or High
);
```

**SOC Scenario:** Baseline deviation analysis

---

## 14. WINDOW FUNCTIONS (Advanced Analytics)

### 14.1 ROW_NUMBER() - Sequential Numbering
```sql
-- Number alerts per user
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

-- Find first alert for each user
SELECT * FROM (
    SELECT 
        InvoiceId,
        CustomerId,
        InvoiceDate,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerId 
            ORDER BY InvoiceDate
        ) AS AlertNum 
    FROM Invoice
) 
WHERE AlertNum = 1;
```

**SOC Scenario:** Tracking attack progression

---

### 14.2 RANK() and DENSE_RANK()
```sql
-- Rank events by size
SELECT 
    Name,
    Bytes,
    RANK() OVER (ORDER BY Bytes DESC) AS SizeRank,
    DENSE_RANK() OVER (ORDER BY Bytes DESC) AS DenseSizeRank 
FROM Track 
LIMIT 20;

-- Rank users by alert count
SELECT 
    c.Email,
    COUNT(i.InvoiceId) AS AlertCount,
    RANK() OVER (ORDER BY COUNT(i.InvoiceId) DESC) AS UserRank 
FROM Customer c
LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId 
GROUP BY c.CustomerId, c.Email 
ORDER BY UserRank;
```

---

### 14.3 COUNT() OVER - Running Statistics
```sql
-- Running count of alerts
SELECT 
    InvoiceId,
    InvoiceDate,
    COUNT(*) OVER (
        ORDER BY InvoiceDate 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS RunningTotal 
FROM Invoice 
ORDER BY InvoiceDate;

-- Count of alerts per user up to each point
SELECT 
    InvoiceId,
    CustomerId,
    InvoiceDate,
    COUNT(*) OVER (
        PARTITION BY CustomerId 
        ORDER BY InvoiceDate
    ) AS AlertCountSoFar 
FROM Invoice;
```

**SOC Scenario:** Trend analysis

---

### 14.4 SUM() OVER - Running Totals
```sql
-- Running total of bytes transferred
SELECT 
    TrackId,
    Name,
    Bytes,
    SUM(Bytes) OVER (
        ORDER BY TrackId
    ) AS CumulativeBytes 
FROM Track 
LIMIT 50;

-- Running total per campaign
SELECT 
    t.TrackId,
    t.AlbumId,
    a.Title AS Campaign,
    t.Bytes,
    SUM(t.Bytes) OVER (
        PARTITION BY t.AlbumId 
        ORDER BY t.TrackId
    ) AS CampaignCumulativeBytes 
FROM Track t
JOIN Album a ON t.AlbumId = a.AlbumId;
```

---

### 14.5 AVG() OVER - Moving Averages
```sql
-- Moving average of alert severity
SELECT 
    InvoiceId,
    InvoiceDate,
    Total,
    AVG(Total) OVER (
        ORDER BY InvoiceDate 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS MovingAvg3 
FROM Invoice 
ORDER BY InvoiceDate;
```

---

### 14.6 Window Functions for Burst Detection
```sql
-- Detect rapid alerts (3+ in short window)
SELECT 
    CustomerId,
    InvoiceId,
    InvoiceDate,
    COUNT(*) OVER (
        PARTITION BY CustomerId 
        ORDER BY InvoiceDate 
        RANGE BETWEEN INTERVAL '5 minutes' PRECEDING AND CURRENT ROW
    ) AS AlertsInLast5Min 
FROM Invoice 
WHERE COUNT(*) OVER (
    PARTITION BY CustomerId 
    ORDER BY InvoiceDate 
    RANGE BETWEEN INTERVAL '5 minutes' PRECEDING AND CURRENT ROW
) >= 3;

-- Note: RANGE INTERVAL not fully supported in SQLite
-- Alternative using ROWS:
SELECT * FROM (
    SELECT 
        CustomerId,
        InvoiceId,
        InvoiceDate,
        COUNT(*) OVER (
            PARTITION BY CustomerId 
            ORDER BY InvoiceDate 
            ROWS BETWEEN 10 PRECEDING AND CURRENT ROW
        ) AS RecentAlerts 
    FROM Invoice
) 
WHERE RecentAlerts >= 5;
```

**SOC Scenario:** Automated attack burst detection

---

### 14.7 LAG() and LEAD() - Time Between Events
```sql
-- Time since previous alert for each user
SELECT 
    CustomerId,
    InvoiceId,
    InvoiceDate,
    LAG(InvoiceDate) OVER (
        PARTITION BY CustomerId 
        ORDER BY InvoiceDate
    ) AS PreviousAlertTime,
    CAST((
        julianday(InvoiceDate) - 
        julianday(LAG(InvoiceDate) OVER (
            PARTITION BY CustomerId 
            ORDER BY InvoiceDate
        ))
    ) * 24 * 60 AS INTEGER) AS MinutesSinceLast 
FROM Invoice;
```

---

## 15. SCHEMA DISCOVERY (Mandatory in Labs)

### 15.1 List All Tables
```sql
-- SQLite system table
SELECT name 
FROM sqlite_master 
WHERE type = 'table' 
ORDER BY name;
```

---

### 15.2 Show Table Structure
```sql
-- Get column information for a table
PRAGMA table_info(Track);

-- More detailed table info
SELECT 
    name AS TableName,
    sql AS CreateStatement 
FROM sqlite_master 
WHERE type = 'table' 
    AND name = 'Track';
```

---

### 15.3 Show All Foreign Keys
```sql
-- Foreign key relationships
PRAGMA foreign_key_list(Track);
PRAGMA foreign_key_list(Invoice);
PRAGMA foreign_key_list(Album);
```

---

### 15.4 Show Indexes
```sql
-- List indexes
SELECT * 
FROM sqlite_master 
WHERE type = 'index';
```

---

### 15.5 Complete Schema Overview
```sql
-- All tables and their structure
SELECT 
    type,
    name,
    tbl_name,
    sql 
FROM sqlite_master 
WHERE type IN ('table', 'index', 'view') 
ORDER BY type, name;
```

**SOC Scenario:** Understanding log database schema before querying

---

## PUTTING IT ALL TOGETHER: COMPLEX SOC QUERIES

### Real-World Investigation Query
```sql
-- Comprehensive threat hunt query
WITH UserRisk AS (
    SELECT 
        c.CustomerId,
        c.Email,
        c.Company,
        COUNT(i.InvoiceId) AS TotalAlerts,
        SUM(CASE WHEN t.GenreId <= 2 THEN 1 ELSE 0 END) AS HighSevAlerts,
        MAX(i.InvoiceDate) AS LastAlert,
        MIN(i.InvoiceDate) AS FirstAlert 
    FROM Customer c
    LEFT JOIN Invoice i ON c.CustomerId = i.CustomerId
    LEFT JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
    LEFT JOIN Track t ON il.TrackId = t.TrackId 
    GROUP BY c.CustomerId, c.Email, c.Company
),
ThreatPattern AS (
    SELECT 
        t.Name AS EventName,
        COUNT(*) AS Frequency,
        AVG(t.Bytes) AS AvgBytes,
        a.Title AS Campaign,
        ar.Name AS ThreatActor 
    FROM Track t
    JOIN Album a ON t.AlbumId = a.AlbumId
    JOIN Artist ar ON a.ArtistId = ar.ArtistId 
    WHERE t.GenreId <= 2 
    GROUP BY t.Name, a.Title, ar.Name 
    HAVING COUNT(*) > 3
)
SELECT 
    ur.Email,
    ur.Company,
    ur.TotalAlerts,
    ur.HighSevAlerts,
    ur.LastAlert,
    CAST((julianday('now') - julianday(ur.LastAlert)) AS INTEGER) AS DaysSinceLastAlert,
    CASE 
        WHEN ur.HighSevAlerts > 10 THEN 'CRITICAL'
        WHEN ur.HighSevAlerts > 5 THEN 'HIGH'
        WHEN ur.TotalAlerts > 10 THEN 'MEDIUM'
        ELSE 'LOW'
    END AS RiskLevel 
FROM UserRisk ur 
WHERE ur.TotalAlerts > 0 
ORDER BY ur.HighSevAlerts DESC, ur.TotalAlerts DESC 
LIMIT 20;
```

---

### Daily SOC Report Query
```sql
-- Daily SOC metrics
SELECT 
    'Last 24 Hours' AS Period,
    COUNT(DISTINCT i.InvoiceId) AS TotalAlerts,
    COUNT(DISTINCT i.CustomerId) AS AffectedUsers,
    COUNT(DISTINCT CASE WHEN t.GenreId = 1 THEN i.InvoiceId END) AS CriticalAlerts,
    COUNT(DISTINCT CASE WHEN t.GenreId = 2 THEN i.InvoiceId END) AS HighAlerts,
    COUNT(DISTINCT CASE WHEN t.GenreId = 3 THEN i.InvoiceId END) AS MediumAlerts,
    SUM(t.Bytes) / 1048576.0 AS TotalDataMB,
    COUNT(DISTINCT t.MediaTypeId) AS UniqueEventTypes,
    COUNT(DISTINCT a.AlbumId) AS ActiveCampaigns 
FROM Invoice i
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId
LEFT JOIN Album a ON t.AlbumId = a.AlbumId 
WHERE i.InvoiceDate >= datetime('now', '-1 day');
```

---

## BEST PRACTICES FOR SOC ANALYSTS

1. **Always start with schema discovery** - PRAGMA table_info()
2. **Use LIMIT** when testing queries on large datasets
3. **Comment your queries** for team knowledge sharing
4. **Build incrementally** - start simple, add complexity
5. **Use CTEs (WITH)** for readable complex queries
6. **Index date/time fields** for performance
7. **Save common queries** as views or scripts
8. **Use EXPLAIN QUERY PLAN** to optimize slow queries
9. **Test WHERE before HAVING** - filter early
10. **Use aliases** for readability

---

## QUICK REFERENCE CHEAT SHEET

**Filtering:** WHERE, AND, OR, NOT, IN, BETWEEN, LIKE, IS NULL
**Aggregation:** COUNT, SUM, AVG, MIN, MAX
**Grouping:** GROUP BY, HAVING
**Joins:** INNER, LEFT, RIGHT, FULL OUTER
**Sorting:** ORDER BY ASC/DESC
**Limiting:** LIMIT, OFFSET
**Subqueries:** WHERE, FROM, SELECT
**Strings:** LOWER, UPPER, SUBSTR, TRIM, REPLACE, LENGTH
**Dates:** strftime, datetime, julianday, date
**Logic:** CASE WHEN THEN END
**Advanced:** Window functions, EXISTS, UNION

---

END OF GUIDE
