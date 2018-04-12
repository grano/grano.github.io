---
layout: post
title:  "Calculating Active Users in SQL"
tags: [sql, wagon, data, data analysis]
---

_[Cross post from the Wagon blog](http://www.wagonhq.com/blog/active-users-in-sql). Thanks to the Wagon team for help with this blog post._

How engaged are your users? How frequently do they visit your website or app? Analytics services like Google Analytics and MixPanel calculate basic counts of daily, weekly, and monthly active users, but it’s difficult to customize or join these results with other data. Writing this query in SQL gives you more control. Let’s do it!

Here’s a table of user logins by day. How many users were active in the last week and month?

<table class="mytable">

<tbody>

<tr>

<th>date</th>

<th>user_id</th>

<th>num_logins</th>

</tr>

<tr>

<td>10/1/15</td>

<td>1</td>

<td>3</td>

</tr>

<tr>

<td>10/1/15</td>

<td>2</td>

<td><em>null</em></td>

</tr>

<tr>

<td>10/1/15</td>

<td>3</td>

<td>1</td>

</tr>

<tr>

<td>10/2/15</td>

<td>1</td>

<td><em>null</em></td>

</tr>

<tr>

<td>10/2/15</td>

<td>2</td>

<td>1</td>

</tr>

<tr>

<td>10/2/15</td>

<td>3</td>

<td>3</td>

</tr>

</tbody>

</table>

Like [calculating running totals](/blog/running-totals-sql), there are two approaches: `self join` or `window function`.

In either approach, it’s helpful to have a table of logins per user for each day, even if the user didn’t login (<em>null</em> in this example). If your data isn’t already organized like this, you can generate a table with a row per day, per user, with the following query (this is Postgres syntax, for other databases, modify the `generate_series` function to generate a table of dates).

{% gist grano/215991d7fb785bf7685a %}

To use this data, you can create a temporary table, use a common table expression, or include it as a subselect.

#### Approach 1: Self Join

A self join is when you join a table with itself. How meta is that? For each row, we ask how many logins that user had in the last week. The join condition requires emails to match and for the date to be within the last 7 days. In line 5, the query sums num_logins for those dates. The case statement identifies the user as active on that day if she had any logins in the prior week.

{% gist grano/a668412b889cb172823f %}

This query generates a table that tells us which users are seven-day-active over time. This result can be aggregated further, filtered for specific dates, used to find inactive users, and joined with other data. In Wagon, we can create a graph of the number of 7 day active users over time.

#### Approach 2: Window Functions

The self join works great, but modern databases have a more efficient way to get the same results. With window functions, we can explicitly aggregate only over rows that we care about with just a single pass through the data. If you have millions or billions of rows (lucky you), the self join will take a long time to compute. In line 5, the query sums num_logins for the user’s previous 14 days. It first partitions the table by email, then evaluates over a set of rows - in this case we’re looking at a specific date range. The case statement classifies the user as active or not just as before.

{% gist grano/a16e64d9164924bc6f3a %}

This query makes it easier to add additional metrics for 7 and 30 day active users. As expected, the wider your definition of active user, the more you’ll have. Use these new powers carefully!
