---
title: "Export query to csv from Oracle Database"
date: 2022-09-10
lastmod: 2022-09-10
draft: false
tags:
  - database
  - sql
---

## Export using SQLPlus prior to 12.2

```sql
-- Query の結果を CSV に出力するための設定
-- Query の出力項目のデータ型が数字の場合、追加の設定が必要かもしれない
SET LINESIZE 2000
SET ECHO OFF
SET FEEDBACK OFF
SET HEADING OFF
SET PAGESIZE 0
SET TIME OFF
SET TIMING OFF
SET TERMOUT OFF
SET TRIMSPOOL ON
SET VERIFY OFF

-- レコードのデータをCSVファイルに出力する
spool result_data.csv

-- ヘーダだけを出力するため
SELECT
    '"COL1"' || ',' ||
    '"COL2"' || ',' ||
    '"COL3"'
FROM DUAL;

-- COL1, COL2, COL3 のデータタイプが文字列の場合
SELECT
    '"' || REPLACE(COL1, '"', '""') || '",' ||
    '"' || REPLACE(COL2, '"', '""') || '",' ||
    '"' || REPLACE(COL3, '"', '""') || '"'
FROM ALL_DATA
;

spool off
```

## Export using SQLPlus 12.2 and above

```sql
set markup csv on
<sql query>
#+end_src

* Export using SQLcl
#+begin_src sql
SET SQLFORMAT csv
SELECT * FROM HR.EMPLOYEES;
```
