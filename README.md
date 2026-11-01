# SQL for Economists — Joins, Windows and the Cardinality Trap

## Objective
Translate common pandas operations into SQL with DuckDB, and show how a one-to-many join can quietly distort statistical results.

## Methodology
- Translated pandas operations (select, filter, sort, groupby, merge) into SQL using DuckDB, and checked that both versions returned the same rows.
- Joined a customers table to an orders table with a LEFT JOIN, then ran the same Welch t-test on revenue before and after the join.
- Used window functions (AVG OVER PARTITION BY, RANK) to attach each region's average and each customer's rank to every row without collapsing the table.
- Tested SQL's three-valued logic by filtering a small survey table with missing incomes using WHERE income != 50000, and compared the result to the same filter in pandas.
- Queried a public Parquet file of about 3 million NYC taxi trips over HTTP without downloading it, reading only the columns the query needed.
- Asked an AI to translate three pandas steps into SQL, revised the prompt once to state how missing incomes should be handled, and compared the row counts of its queries against the pandas results.

## Key Findings
- **The cardinality trap:** The one-to-many LEFT JOIN turned 600 customer rows into 3,612 rows. The same Welch t-test on the same revenue figures went from t = 0.07 (p = 0.9413) to t = 0.71 (p = 0.4749). The repeated rows made the test act as if it had six times the data, which shrank the standard error and shifted the difference toward customers with more orders.
- **Window functions:** Group averages and rankings were added to all 600 rows without losing any of them, unlike GROUP BY, which collapses each group into one row.
- **NULL handling:** WHERE income != 50000 kept 3 of 5 survey rows, because SQL treats a comparison with a missing value as unknown and drops those rows. The same filter in pandas kept all 5.
- **Remote data:** DuckDB answered a query on about 3 million taxi trips without downloading the full file, which means the real limit is how much data a query has to read, not how big the file is.
- **AI translation:** After the prompt was revised to say that missing incomes should be kept, all three AI queries matched the pandas row counts. Checking row counts caught a difference that the queries alone would not have shown.
