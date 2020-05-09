SQL STYLE GUIDE
===============

This is yet another SQL style guide because I find other maybe less adapted to my needs, there will not be a perfect coding style anyway.

You can find another different stle guide [here](https://www.sqlstyle.guide/) the idea is to pick one a to stick to it alongside your developments.

This guide is released under [Creative Common SA](http://creativecommons.org/licenses/by-sa/4.0/)

```PLpgSQL
with recursive
  fibo (level, n, m) as (
    select 0::int8 as level, 0::int8 as n, 1::int8 as m
    union all
    select
      parent.level + 1      as level,
      parent.m              as n,
      parent.m + parent.n   as m
    from fibo as parent
    where parent.level < 20
  )
select level, n, m from fibo
;
```

Conventions
===========

 * use lower case
 * use [snake case](https://en.wikipedia.org/wiki/Snake_case)
 * use ISO-8601 date style (YYYY-mm-dd HH:ii:ss.uuuu( ±00:00))
 * use the same indentation style everywhere (spaces or tabs)

Naming
======

 * since a table defines a type, it must be singular name unless a single record represents a collection
 * columns are singular names unless they are arrays types
 * use an underscore where you expect an uppercase in your upper level code class (Ex: `v_i_p` → `VIP`)
 * avoid as much as possible using SQL keyword as names (no, `user` is not a good name for a table)
 * column names may be prefixed or suffixed:
    * `is_something` is a boolean
    * `something_at` is a date
    * `something_id` is a primary key

Inline or expanded
==================

List definitions like fields of a `select`, conditions, sort criteria or joined tables may be either declared on the same line or one item per line. Whatever the style chosen, it must be the same for the whole list. Avoid long inline lists exceeding 80 chars.

Good:

```sql
select one, twenty_two as three, count(four) as fours…
select
  one,
  twenty_two as two,
  count(four) as fours
  case
    when five = 0 then 'zero'
    else 'something'
  end as five,
```

Avoid:

```sql
select one, twenty_two as three, count(four) as fours, case when five = 0 then 'zero' else 'something' end as five
-- ↑ this line is too long
select one, twenty_two as three
  count(fours) as four, case
    when five = 0 then 'zero'
    else 'something'
  end as five,
-- ↑ this line mixes inline and expanded style
```

Code blocks and indentation
===========================

A SQL query is divided in several parts that must be easily identifiable. The query blocks are the following:
`with` sub queries & main queries. Queries are on the same level as `union`, `intersect` etc.

Queries as such are divided in the following parts:
from / where / group by / having / select / window / order by / limit. These blocks are indented at the same level.

```sql
select student_id, first_name, last_name, age, count(examination) as count_exam
from student
  left outer join examination
    using student_id
where student.age >= 16
  and (
    examination is null
    or not examination.is_success
  )
  and student.gender = 'M'
group by student_id, first_name, last_name, age
order by count_exam desc
limit 10
```

# select

The select statement is the first part of SQL queries and it defines the output projection of the query.
If the fields are expanded on the lines below, the `select` keyword must be alone with `all` or `distinct` keywords if any:

```sql
select distinct on (report_day, report_count)
  report_day,
  report_count,
  …
```

## aliases and type casting

There are no enforcement about column aliasing except that all fields must have a unique and predictable name:

    * explicitly alias and cast anonymous fields: `select 0::integer as start`
    * explicitly alias function calls: `select count(*) as total_count`
    * explicitly alias conditional expressions: `select case … end as my_value`


In some cases, it may be more readable to tabulate aliases:

```sql
select
  frstnm    as first_name,
  lstnm     as last_name,
  dob       as birthdate
from
  student
```

Be aware maintaining this may become tedious with a growing list of fields as some field definition may exceed the chosen alias position:

```sql
select
  frstnm    as first_name,
  lstnm     as last_name,
  dob       as birthdate,
  count(examination)
            as exam_total,
  count(examination) filter (examination.is_success)
            as exam_success,
  count(examination) filter (not examination.is_success)
            as exam_failed,
from
  student
```

When using this technique all fields must be aliased to avoid gaps in the fields list.
The example below:

```sql
select
  frstnm as first_name,
  lstnm as last_name,
  dob as birthdate,
  count(examination) as exam_total,
  count(examination) filter (examination.is_success) as exam_success,
  count(examination) filter (not examination.is_success) as exam_failed,
from
  student
```

is also fine.

## conditional values `case when`

`case when` definition may be inline if the line does not exceed 80 chars. It is advised to indent such blocks though:

```sql
    select
      case when age >= 18 then 'over 18'::text else 'under 18'::text end as majority,
```

is ok but not as good as:

```sql
select
  case
    when age >= 18 then 'over 18'::text
    else 'under 18'::text
  end as majority
```

## window definitions

Window definitions may be also be inline or expanded:

```sql
select
  departement_name,
  employee_id,
  salary,
  rank() over (partition by departement_name order by salary desc) as ranking
```

is as good as

```sql
select
  departement_name,
  employee_id,
  salary,
  rank() over (
    partition by departement_name
    order by salary desc
  ) as dept_salary_ranking
```
