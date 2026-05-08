---
layout: article
title: "Finding Duplicate Addresses Using the Levenshtein Distance Metric in SQL"
date: 2018-08-30
author: Carlos Blanco
categories: [sql, algorithms]
tags: [sql, postgresql, levenshtein, string-matching]
original_site: Nearsoft
original_url: https://nearsoft.com/blog/finding-duplicate-addresses-using-the-levenshtein-distance-metric-in-sql/
canonical_url: https://nearsoft.com/blog/finding-duplicate-addresses-using-the-levenshtein-distance-metric-in-sql/
---

We've all been in the situation where the `LIKE` operator in SQL isn't good enough for the needs of the task at hand. The `LIKE` operator lacks the ability to find strings that are similar but not quite the same. This is when the Levenshtein distance algorithm comes in handy.

## What is the Levenshtein distance?

The Levenshtein distance is a metric that measures the difference between two strings — specifically, the minimum number of single-character edits (insertions, deletions, substitutions) required to change one string into the other. For example, the distance between `"books"` and `"back"` is three:

```
books -> baoks  (substitution: "o" → "a")
baoks -> backs  (substitution: "o" → "c")
backs -> back   (deletion of "s")
```

Phil Factor writes about the edit distance and gives an implementation in MS SQL Server's T-SQL in his post [String Comparisons in SQL: Edit Distance and the Levenshtein algorithm](https://www.red-gate.com/simple-talk/databases/sql-server/t-sql-programming-sql-server/string-comparisons-in-sql-edit-distance-and-the-levenshtein-algorithm/). Other databases ship with a built-in: `LEVENSHTEIN` in PostgreSQL, `EDIT_DISTANCE_SIMILARITY` in Oracle, and `EDITDIST3` in SQLite.

This is very useful when trying to identify that `931 Main St` is the "same" as `931 Main Street` — a common problem in any system that stores client information. The trick is to convert the raw distance into a ratio based on the length of the longer string, giving you a percentage of similarity. In my experience, a ratio below 50% works well as a threshold.

## An example in PostgreSQL

Say we have this table:

```sql
CREATE TABLE Addresses(
  LINE1 text,
  LINE2 text
);
```

With sample data where each row holds two versions of the same address — similar enough for a human, but not for `LIKE`:

```sql
INSERT INTO Addresses (LINE1, LINE2) VALUES
  ('9128 LEEWARD CIR, INDIANAPOLIS, IN',      '9128 Leeward Circle, Indianapolis, IN'),
  ('101 OCEAN LANE DRIVE, KEY BISCAYNE, FL',  '101 Ocean Lane Drive Unit 1010, Key Biscayne, FL'),
  ('9301 EVERGREEN DRIVE, PARMA, OH',          '9301 Evergreen Dr, Parma, OH'),
  ('1817 BERTRAND DR, LAFAYETTE, LA',          '2924 Polo Ridge Ct, Charlotte, NC'),
  ('201 E 87TH ST, NEW YORK, NY',              '201 E 87th Street 3E, New York, NY'),
  ('799 CARRIGAN AVE, OVIEDO, FL',             '799 Carrigan Avenue, Oviedo, FL'),
  ('4014 CADDIE DRIVE, ACWORTH, GA',           '10617 WEYBRIDGE DR, TAMPA, FL');
```

Now use `LEVENSHTEIN` to calculate a similarity ratio:

```sql
SELECT
  Line1,
  Line2,
  LEVENSHTEIN(UPPER(line1), UPPER(line2)) AS distance,
  LEVENSHTEIN(UPPER(line1), UPPER(line2))::decimal
    / GREATEST(length(line1), length(line2)) AS ratio
FROM Addresses;
```

| # | line1 | line2 | distance | ratio |
|---|-------|-------|----------|-------|
| 1 | 9128 LEEWARD CIR, INDIANAPOLIS, IN | 9128 Leeward Circle, Indianapolis, IN | 3 | 0.08… |
| 2 | 101 OCEAN LANE DRIVE, KEY BISCAYNE, FL | 101 Ocean Lane Drive Unit 1010, Key Biscayne, FL | 10 | 0.20… |
| 3 | 9301 EVERGREEN DRIVE, PARMA, OH | 9301 Evergreen Dr, Parma, OH | 3 | 0.09… |
| 4 | 1817 BERTRAND DR, LAFAYETTE, LA | 2924 Polo Ridge Ct, Charlotte, NC | 23 | 0.69… |
| 5 | 201 E 87TH ST, NEW YORK, NY | 201 E 87th Street 3E, New York, NY | 7 | 0.20… |
| 6 | 799 CARRIGAN AVE, OVIEDO, FL | 799 Carrigan Avenue, Oviedo, FL | 3 | 0.09… |
| 7 | 4014 CADDIE DRIVE, ACWORTH, GA | 10617 WEYBRIDGE DR, TAMPA, FL | 22 | 0.73… |

Rows 4 and 7 are clearly different addresses (ratio > 0.5). Adding a `WHERE` clause filters them out:

```sql
SELECT
  Line1,
  Line2,
  LEVENSHTEIN(UPPER(line1), UPPER(line2)) AS distance,
  LEVENSHTEIN(UPPER(line1), UPPER(line2))::decimal
    / GREATEST(length(line1), length(line2)) AS ratio
FROM Addresses
WHERE LEVENSHTEIN(UPPER(line1), UPPER(line2))::decimal
        / GREATEST(length(line1), length(line2)) < 0.5;
```

| # | line1 | line2 | distance | ratio |
|---|-------|-------|----------|-------|
| 1 | 9128 LEEWARD CIR, INDIANAPOLIS, IN | 9128 Leeward Circle, Indianapolis, IN | 3 | 0.08… |
| 2 | 101 OCEAN LANE DRIVE, KEY BISCAYNE, FL | 101 Ocean Lane Drive Unit 1010, Key Biscayne, FL | 10 | 0.20… |
| 3 | 9301 EVERGREEN DRIVE, PARMA, OH | 9301 Evergreen Dr, Parma, OH | 3 | 0.09… |
| 5 | 201 E 87TH ST, NEW YORK, NY | 201 E 87th Street 3E, New York, NY | 7 | 0.20… |
| 6 | 799 CARRIGAN AVE, OVIEDO, FL | 799 Carrigan Avenue, Oviedo, FL | 3 | 0.09… |

You can play with this yourself in [SqlFiddle](http://sqlfiddle.com/).
