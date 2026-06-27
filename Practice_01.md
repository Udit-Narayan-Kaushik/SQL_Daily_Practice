# Calculate Average Session Time per User

## Problem

Calculate each user's average session time, where a session is defined as the time difference between a page_load and a page_exit. 
Assume each user has only one session per day. 
If there are multiple page_load or page_exit events on the same day, use only the latest page_load and the earliest page_exit. 
Only consider sessions where the page_load occurs before the page_exit on the same day. Output the user_id and their average session time.

---

## Approach

```
Raw events
      │
      ▼
Group by user + date
      │
      ▼
Latest page_load (MAX)
Earliest page_exit (MIN)
      │
      ▼
Discard invalid sessions
(page_load >= page_exit)
      │
      ▼
Compute session duration
      │
      ▼
Average duration per user
```

### Key SQL Concepts

- `GROUP BY` to create one session per user per day.
- `MAX(CASE WHEN ...)` to get the latest `page_load`.
- `MIN(CASE WHEN ...)` to get the earliest `page_exit`.
- `TIMESTAMPDIFF()` to calculate the session duration.
- `AVG()` to compute the average session time for each user.

---

## Solution

```sql
WITH sessions AS (
    SELECT
        user_id,
        DATE(timestamp) AS session_day,
        MAX(
            CASE
                WHEN action = 'page_load'
                THEN timestamp
            END
        ) AS load_time,
        MIN(
            CASE
                WHEN action = 'page_exit'
                THEN timestamp
            END
        ) AS exit_time
    FROM facebook_web_log
    GROUP BY
        user_id,
        DATE(timestamp)
)

SELECT
    user_id,
    AVG(TIMESTAMPDIFF(SECOND, load_time, exit_time)) AS avg_session_time
FROM sessions
WHERE load_time < exit_time
GROUP BY user_id;
```

---

## Time Complexity

- Building the CTE: **O(n)**
- Final aggregation: **O(n)**

Overall complexity: **O(n)**

---

## Takeaway

This problem demonstrates the **conditional aggregation** pattern:

```sql
MAX(CASE WHEN condition THEN value END)
MIN(CASE WHEN condition THEN value END)
```

This is a common SQL interview technique used to extract different aggregate values from the same group.
