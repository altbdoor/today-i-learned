The Evil that is Offset in MySQL
===

November 17, 2014

Recently, I am working on a project which requires pagination. The first thing to come to mind is the use of MySQL's `OFFSET`. Little did I know, that `OFFSET` was a really expensive process.

The common knowledge of pagination is to use `LIMIT` to limit the number of results, and `OFFSET` to filter results based on the page number. Concerned about the performance, I decided to search information regarding this combo. Looks like I was late to the party.

> An *EXPLAIN* shows, that 100015 rows were read but only 15 were really needed and the rest was thrown away. Large offsets are going to increase the data set used, MySQL has to bring data in memory that is never used!
> 
> &mdash; [Robert Eisele, Optimized Pagination using MySQL](http://www.xarg.org/2011/10/optimized-pagination-using-mysql/)

<!-- -->
> MySQL has to find all 50,000 rows, step over the first 49,990, then deliver the 10 for that distant page.
> 
> If it is a crawler ('spider') that read all the pages, then it actually touched about 125,000,000 items to read all 5,000 pages. 
> 
> Reading the entire table, just to get a distant page, can be so much I/O that it can cause timeouts on the web page. Or it can interfere with other activity, causing other things to be slow. 
> 
> &mdash; [Rick James, Pagination](http://mysql.rjweb.org/doc.php/pagination)

<!-- -->
> MySQL LIMIT may return results way too slow when the offset is high. When used in a table with a lot of rows this query gets slower and slower the higher page offset gets. It seems like for some reason MySQL is operating on the whole table data instead of working only with the rows we'll actually need.
> 
> &mdash; [devoluk, MySQL LIMIT offset performance](http://devoluk.com/mysql-limit-offset-performance.html)

That's enough to spook me. The fact is, the project I am working on is small, but in due time, its a ticking bomb. So I started looking for ways to optimize this query. A common result found is to use `INNER JOIN` or other forms of `JOIN`.

[One question in StackExchange](http://dba.stackexchange.com/questions/55435/using-offset-in-a-large-table-by-a-simple-sub-query) particularly caught my attention, and explains pretty much what I need to know. Give it a read.

At the end of the day, I tested it out with [MySQL's sakila database](http://dev.mysql.com/doc/index-other.html). Specifically, the rental table, which has approximately 16,000 rows. The query in comparison here are:

```sql
-- normal offset
SELECT SQL_NO_CACHE rental_id,rental_date, return_date FROM rental
	WHERE staff_id=1
	ORDER BY rental_id DESC
	LIMIT 5 OFFSET 8000

-- left join
SELECT SQL_NO_CACHE self2.rental_id, self2.rental_date, self2.return_date FROM (
	SELECT rental_id FROM rental
		WHERE staff_id=1
		ORDER BY rental_id DESC
		LIMIT 5 OFFSET 8000
) AS self1 LEFT JOIN rental AS self2 USING (rental_id)
```

Notice the use of `SQL_NO_CACHE`. It prevents the query from being cached. Using an increment of 1,000 with the average of five takes, these are the results.

#### Left Join (seconds)

| | Take 1 | Take 2 | Take 3 | Take 4 | Take 5 |
| --- | --- | --- | --- | --- | --- |
| 1000 | 0.0011 | 0.0012 | 0.0011 | 0.0013 | 0.0011 |
| 2000 | 0.0017 | 0.0019 | 0.0019 | 0.0018 | 0.0018 |
| 3000 | 0.0025 | 0.0024 | 0.0024 | 0.0024 | 0.0025 |
| 4000 | 0.0031 | 0.0030 | 0.0029 | 0.0029 | 0.0048 |
| 5000 | 0.0037 | 0.0038 | 0.0037 | 0.0036 | 0.0037 |
| 6000 | 0.0045 | 0.0065 | 0.0042 | 0.0042 | 0.0042 |
| 7000 | 0.0050 | 0.0049 | 0.0050 | 0.0051 | 0.0049 |
| 8000 | 0.0062 | 0.0072 | 0.0054 | 0.0056 | 0.0055 |

#### Normal Offset (seconds)

| | Take 1 | Take 2 | Take 3 | Take 4 | Take 5 |
| --- | --- | --- | --- | --- | --- |
| 1000 | 0.0021 | 0.0024 | 0.0033 | 0.0021 | 0.0020 |
| 2000 | 0.0036 | 0.0036 | 0.0045 | 0.0036 | 0.0037 |
| 3000 | 0.0053 | 0.0052 | 0.0055 | 0.0053 | 0.0051 |
| 4000 | 0.0067 | 0.0072 | 0.0074 | 0.0067 | 0.0068 |
| 5000 | 0.0086 | 0.0084 | 0.0107 | 0.0117 | 0.0085 |
| 6000 | 0.0099 | 0.0099 | 0.0098 | 0.0099 | 0.0129 |
| 7000 | 0.0115 | 0.0116 | 0.0174 | 0.0115 | 0.0152 |
| 8000 | 0.0133 | 0.0155 | 0.0157 | 0.0131 | 0.0130 |

#### Averaged (seconds)
| | Left Join | Normal Offset |
| --- | --- | --- |
| 1000 | 0.0012 | 0.0024 |
| 2000 | 0.0018 | 0.0038 |
| 3000 | 0.0024 | 0.0053 |
| 4000 | 0.0033 | 0.0070 |
| 5000 | 0.0037 | 0.0096 |
| 6000 | 0.0047 | 0.0105 |
| 7000 | 0.0050 | 0.0134 |
| 8000 | 0.0060 | 0.0141 |

Please note that the tests here are done in PHPMyAdmin. For what it's worth, your mileage may vary and this whole test could just be plain wrong, as compared to a [comprehensive test by devoluk](http://devoluk.com/mysql-limit-offset-performance.html).

---

#### References
- http://www.xarg.org/2011/10/optimized-pagination-using-mysql/
- http://mysql.rjweb.org/doc.php/pagination
- http://devoluk.com/mysql-limit-offset-performance.html
- http://dba.stackexchange.com/questions/55435/using-offset-in-a-large-table-by-a-simple-sub-query
