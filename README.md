POMM SQL STYLE GUIDE
====================

This document is in version: **DEVELOPMENT**

This guide is released under [Creative Common SA-BY](http://creativecommons.org/licenses/by-sa/4.0/) and will be referred as PSSG in this document.


I use mainly PostgreSQL and I like [Dimitri Fontaine](https://tapoueh.org/)'s approach of SQL coding: abandoning the old fashioned upper case SQL keywords to focus on the expressiveness of the language.
This is yet another SQL style guide because I find others maybe less adapted to my needs: embedding SQL in PHP code using the [Pomm library](http://www.pomm-project.org). This style guide can also be used for generic SQL development, there will not be a perfect coding style anyway. You can find another different style guide [here](https://www.sqlstyle.guide/) the idea is to pick one and to stick to it.

Here is an example of how a query could look like according to this style guide:

```PLpgSQL
with recursive
    fibo (level, n, m) as (
        select 0::int8 as level, 0::int8 as n, 1::int8 as m
        union all
        select
            previous.level + 1       as level,
            previous.m               as n,
            previous.m + previous.n  as m
        from fibo as previous
        where previous.level < 20
    )
select
    level,
    m as fibo,
    sum(m) over (rows 5 preceding) as rabbits_alive
from fibo
```

This guide
==========

In order to be able to create an automatic parser that states if an existing piece of code conforms to this style guide, it will use the following rules:

 * Some conventions MUST be respected, if not it is an error hence the coding style is NOT conform to this document.
 * Some conventions MUST be respected UNLESS an exception is described. In this case it is perfectly fine to write code as expressed in the exception.
 * Some conventions MUST be respected AS MUCH AS POSSIBLE. If it is not possible, the convention is invalid and MUST NOT be enforced at all. A warning MUST be raised but it NOT an error not to conform.
 * Some conventions SHOULD be respected, however it is not an error to locally contravene the rule, a information note may be issued.
 * Some conventions MAY be used but there are no enforcements to do so.
 * Some conventions do not have exceptions and this indication can be clearly given at once with the word ONLY.
 * Some hints are given and must not interfere with code validation.

As soon as a SQL code respects the constraints described in this document, it can be stated as conform but an indication of the version of this coding style MUST be indicated with the statement:

    Conforms PSSG version X.Y.Z

The version will follow semver rules:

 * A new patch is released for typographic errors or disambiguations. No constraints will be added or removed in patches.
 * A new minor is released if constraints are added or adapted but a code conforming a higher version must also conform lower versions. 
 * A new major is released when rules change and this is considered as a new work independent from other major releases.

Conventions
===========

 * The code MUST use lower case ONLY.
 * The code MUST use [snake case](https://en.wikipedia.org/wiki/Snake_case) names.
 * The code MUST use ISO-8601 date style (YYYY-mm-dd HH:ii:ss.uuuu( ±00:00)).
 * The code MUST use the same indentation style everywhere (spaces or tabs).
 * The code lines SHOULD not exceed 80 chars.
 * The code lines MUST NOT exceed 120 chars AS MUCH AS POSSIBLE.

Naming
======

 * All names MUST be lower case ONLY.
 * All names MUST use snake case ONLY.
 * Since a table defines a type, it MUST be singular name UNLESS a record represents a collection.
 * Columns MUST be singular names UNLESS they are collection type (array, HStore etc.).
 * SQL objects MUST NOT use SQL keyword as name AS MUCH AS POSSIBLE.
 * Hint: use an underscore where you expect an uppercase in your upper level code class (Ex: `v_i_p` → `VIP`).
 * Hint: avoid as much as possible using SQL keyword as names (no, `user` is not a good name for a table).
 * Column names MAY be prefixed or suffixed:
    * `is_something` is a boolean
    * `something_at` is a date
    * `something_id` is a primary key

Inline or expanded
==================

List definitions like fields of a `select`, conditions, sort criteria or joined tables MUST be either declared on the same line or one item per line. Whatever the style chosen, it MUST be the same for the whole list. Avoid long inline lists exceeding 80 chars.

Good:

```PLpgSQL
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

```PLpgSQL
select one, twenty_two as three, count(four) as fours, case when five = 0 then 'zero' else 'something' end as five
-- ↑ this line is too long
```

This is an error:

```PLpgSQL
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

```PLpgSQL
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

```PLpgSQL
select distinct on (report_day, report_count)
  report_day,
  report_count,
  …
```

It may happen a long expression makes the line over 80 or even 120 characters. It is fine to break the line with an indentation:

```PLpgSQL
select distinct on (report_day, report_count)
  report_day,
  (report_count * 100 / ( report_a + report_b + report_c))
    - (previous_report * 100 / ( report_a + report_b + report_c))
    as total_report_percent,
```

This obviously cannot be used with the inline format, it requires the expanded list format.

## aliases and type casting

There are no enforcement about column aliasing except that all fields MUST have a unique and predictable name:

    * anonymous fields MUST be aliased and cast: `select 0::integer as start`
    * function calls MUST be explicitly aliased: `select count(*) as total_count`
    * conditional expressions MUST be explicitly aliased: `select case … end as my_value`
    * values in conditional expressions MUST also be cast: 

```PLpgSQL
select
  case
    when revenues >= 1000 then calculate(revenues)
    else 0::float8
  end as taxes,
…
```

Name aliasing is not mandatory unless alias are tabulated:

```PLpgSQL
select
  frstnm    as first_name,
  lstnm     as last_name,
  dob       as birthdate
from
  student
```

Be aware maintaining this may become tedious with a growing list of fields as some field definition may exceed the chosen alias position:

```PLpgSQL
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

When using this technique all fields MUST be aliased to avoid gaps in the fields list.
The example below:

```PLpgSQL
select
  frstnm as first_name,
  lstnm as last_name,
  dob as birthdate,
  count(examination) as exam_total,
  count(examination) filter (where examination.is_success)
    as exam_success,
  count(examination) filter (where not examination.is_success)
    as exam_failed,
from
  student
```

is also fine.

## conditional values `case when`

`case when` definition may be inline if the line does not exceed 80 chars. It is advised to indent such blocks though:

```PLpgSQL
    select
      case when age >= 18 then 'over 18'::text else 'under 18'::text end as majority,
    …
```

is ok but not as good as:

```PLpgSQL
select
  case
    when age >= 18 then 'over 18'::text
    else 'under 18'::text
  end as majority
  …
```

## window definitions

Window definitions MUST be also be inline or expanded:

```PLpgSQL
select
  departement_name,
  employee_id,
  salary,
  rank() over (partition by departement_name order by salary desc) as ranking
  …
```

is as good as

```PLpgSQL
select
  departement_name,
  employee_id,
  salary,
  rank() over (
    partition by departement_name
    order by salary desc
  ) as dept_salary_ranking
  …
```

Note in the case the definition is expanded the opening parenthesis MUST be on the same line as the `over` clause and the closing parenthesis MUST be on the same line as the alias.

## aggregates or window filtering

The `filter` clause of the aggregate MUST be on the same line as the aggregate function it relates to AS MUCH AS POSSIBLE

```PLpgSQL
select
  student.first_name,
  student.last_name,
  count(examination) as total_exams,
  count(examination) filter (where examination.is_success) as exams_succeeded
  …
```

Window functions may also use the `filter` clause, the rules described above apply:

```PLpgSQL
select
  sales * 100 / sum(sales) filter (where sales > 0) over (
    partition by departement
  ) as departement_percent_sales,
  …
```

