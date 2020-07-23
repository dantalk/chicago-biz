# Painlessly moving from CSV to SQLite

<!--ts-->
   * [Painlessly moving from CSV to SQLite](csv2sqlite.md#painlessly-moving-from-csv-to-sqlite)
      * [Intro](csv2sqlite.md#intro)
      * [Tools and concepts](csv2sqlite.md#tools-and-concepts)
      * [Using csvs-to-sqlite](csv2sqlite.md#using-csvs-to-sqlite)
      * [SQL vs ripgrep/xsv hacks](csv2sqlite.md#sql-vs-ripgrepxsv-hacks)
         * [Creating a list/view of active licenses](csv2sqlite.md#creating-a-listview-of-active-licenses)
         * [Creating a list/view of tobacco-related licenses](csv2sqlite.md#creating-a-listview-of-tobacco-related-licenses)
         * [Group count by license_description](csv2sqlite.md#group-count-by-license_description)
         * [Group count by license by year](csv2sqlite.md#group-count-by-license-by-year)

<!-- Added by: dan, at: Wed Jul 22 23:33:06 CDT 2020 -->

<!--te-->



## Intro 

This is a quickie guide for a very general best practice: moving CSV data into SQLite. As fun as it is to data munge via a dozen piped Unix commands at the command line, some data work ends up being much simpler and expressive and efficient in SQL. 

From here on out, I'll try to show how to do various data wrangling tasks in SQL and in command-line hackery of ripgrep/xsv... 

## Tools and concepts

- [SQLite](https://sqlite.org/index.html): don't really have the time to get into SQL in-depth, since it's a whole damn language. But SQLite is definitely my recommendation for lightweight and powerful data querying.   
- [csvs-to-sqlite](https://github.com/simonw/csvs-to-sqlite): a handy command-line tool that vastly expedites importing CSVs into a SQLite database; installable via simple `pip install csvs-to-sqlite`
- [DB Browser](https://sqlitebrowser.org/): even though I'll try to use the command-line and spreadsheet for most of this walkthrough, DB Browser is an amazing cross-platform and open source client for SQLite. It's designed to bridge the gap between Excel and SQL databases, so I often find it more useful for browsing data than a spreadsheet. 



## Using csvs-to-sqlite

```sh
$ csvs-to-sqlite licenses.csv \
    -i 'account_number,site_number' \
    -i 'license_id' -i 'date_issued' \
    -i 'license_term_start_date' -i 'license term expiration_date' \
    -i 'license_description' \
    biz.sqlite 
```


## SQL vs ripgrep/xsv hacks

### Creating a list/view of active licenses

### Creating a list/view of tobacco-related licenses



### Group count by license_description

In SQL:

```sql
SELECT 
    license_description
    , COUNT(1) AS n
FROM licenses
GROUP BY license_description
ORDER BY n DESC;
```



### Group count by license by year




