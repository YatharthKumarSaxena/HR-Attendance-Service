# DECISIONS.md

1. **Indexes.**
We created a unique index on `employees: {emp_code: 1}` for employee uniqueness and lookups, plus `employees: {joined_on: 1}` and `{department: 1, joined_on: 1}` to eliminate COLLSCAN during headcount aggregations. On `attendance_logs`, we created a unique compound index `{emp_code: 1, date: 1}` to guarantee exactly one record per employee-day, `{emp_code: 1, punch_in: -1}` to instantly find open punch-ins across midnight, `{date: -1, emp_code: 1}` for paginated list queries, and `{date: 1, late_minutes: 1}` for leaderboard filtering. We considered and rejected standalone indexes on `status` because low cardinality yields poor selectivity and unnecessary write overhead.

2. **Punch-in race.**
Both concurrent requests validate the employee and calculate the identical attendance date, then attempt `insert_one`. MongoDB's unique index `{emp_code: 1, date: 1}` atomically serializes the inserts: exactly one operation acquires the slot and returns 201 Created, while the second triggers `DuplicateKeyError`. Our service catches this exception and returns 409 Conflict.

3. **Ties.**
We use MongoDB's `$rank` window function for standard competition ranking (1, 2, 2, 4). When employees tie at the cutoff rank $\le \text{limit}$, all tied employees are returned because the filter is `rank <= limit`. Ties are ordered deterministically by `emp_code` ascending.

4. **Headcount.**
The pipeline starts from the `employees` collection filtered by `joined_on <= month_end`, groups by `department` to compute total headcount, and uses `$lookup` to attach matching logs. Employees with zero logs are retained in the group with an empty logs array, correctly contributing to headcount.

5. **100x Scale.**
At 100x data (~10 million records), we would maintain pre-aggregated monthly summary and daily trend collections updated asynchronously via MongoDB change streams or event-driven workers, avoiding runtime cross-collection aggregations.
