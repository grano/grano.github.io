---
layout: post
title:  "Calculating Running Totals using SQL"
tags: [sql, wagon, data, data analysis, postgres]
---

_[Cross post from the Wagon blog](http://www.wagonhq.com/blog/running-totals-sql). Thanks to the Wagon team for help with this blog post._

How many users joined in the last 5 months? What were total sales in Q2? How much revenue came from the March sign up cohort?

Although these questions can be answered with a single number, it can be useful to see a _running total_ over time: how many unique users joined, or how much cumulative revenue was received by day over some period.

Usually, data is stored incrementally. For example, here’s a table of sales per day:

<table class="table">

<tbody>

<tr>

<th>Date</th>

<th>Sales</th>

</tr>

<tr>

<td style="width: 216px;">10/1/2015</td>

<td>5</td>

</tr>

<tr>

<td>10/2/2015</td>

<td>3</td>

</tr>

<tr>

<td>10/3/2015</td>

<td>7</td>

</tr>

<tr>

<td>10/4/2015</td>

<td>8</td>

</tr>

<tr>

<td>10/5/2015</td>

<td>2</td>

</tr>

<tr>

<td>10/6/2015</td>

<td>3</td>

</tr>

<tr>

<td>10/7/2015</td>

<td>6</td>

</tr>

</tbody>

</table>

How do we generate the following table of cumulative sales over time? In SQL, there are two typical approaches: a self join or a window function.

<table class="table">

<tbody>

<tr>

<th>Date</th>

<th>Running Total of Sales</th>

</tr>

<tr>

<td style="width: 216px;">10/1/2015</td>

<td>5</td>

</tr>

<tr>

<td>10/2/2015</td>

<td>8</td>

</tr>

<tr>

<td>10/3/2015</td>

<td>15</td>

</tr>

<tr>

<td>10/4/2015</td>

<td>23</td>

</tr>

<tr>

<td>10/5/2015</td>

<td>25</td>

</tr>

<tr>

<td>10/6/2015</td>

<td>28</td>

</tr>

<tr>

<td>10/7/2015</td>

<td>34</td>

</tr>

</tbody>

</table>

A self join is a query that compares a table to itself. In this case, we’re comparing each date to any date less than or equal to it in order to calculate the running total. Concretely, we take the sum of `sales` in the second table over every row that has a date less than or equal to the date coming from the first table. This is Postgres/Redshift syntax, but other SQL dialects are very similar.

{% gist grano/b705f532374c0ec02f03 %}

This is not a bad approach; it is a nice showcase of how extensible SQL can be using only `select`, `from`, `join`, and `group by` statements.

But it is a lot of code for a simple task. Let’s try a window function. They are designed to calculate a metric over a set of rows. In our case, we want to sum every row where the date is less than or equal to the date in the current row.

{% gist grano/88fcf67e5ff14ae9e1c2 %}

The window function can filter and arrange the set of rows to run the function over. Here the `order by date rows unbounded preceding` limits the sum function to only `sales` before the date of the current row. Window functions are incredibly useful for time-based analytical queries; to learn more, the [Postgres docs](http://www.postgresql.org/docs/9.4/static/tutorial-window.html) are a great place to start.

The final step of creating a chart and sharing it triumphantly with your teammates is easily accomplished using Wagon. Window functions for the win!
