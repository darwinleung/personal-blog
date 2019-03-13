---
title: "Modelling Snowplow event data notes"
date: 2018-09-01T12:00:00-04:00
draft: false
tags: [ "snowplow" , "sql", "bigquery"]
categories: [ "programming", "data processing", "feature extraction", "analytics"]

---

# Modelling Snowplow event data notes

Goals:

1. Modelling pageviews level metric (timeonpage, scroll depth)
2. Modelling sessions level metric (session count, session duration)

Note:  All queries in this doc are *NOT* tested or validated, they are based on assumptions and my guess only for high level understanding only.  Please always consult Snowflake and make modification accordingly before using them for production.



## Validating readily available metrics

First, we would like to see if the pageviews count collected from Snowplow match roughly with the data we collected from GA:

```sql
  # pageviews count by date
SELECT
  DATE(collector_tstamp) AS Date,
  COUNT(*) AS Pageviews
FROM
  `snowplow.snowplow.events`
WHERE
  DATE(collector_tstamp) >= "2018-08-01"
  AND event = 'page_view'
GROUP BY
  Date
ORDER BY
  Date;
```



# Modelling Pageviews level metric

## Finding Time On Page 

ref: https://www.snowflake-analytics.com/blog/data-modeling/modelling-pageviews/

They find Time on page by joining a web_page table with events table and group by the `pageview_id` from the web_page table. The `pageview_id` in the web_page table is generated by automatically creating a UUID for each pageview which is attached to all events within the pageviews that subsequently shows up in the table. In our case, we don't have the web_page table as well as `pageview_id` to group by. However, we can try to deduce the same information by defining our own `pageviews_id`. Or try to group by user + sessions + url, then to count the page ping with the same info. 

In our current setting, I believe each pageview event represent the first 30 secs, and each subsequent pageping track 10 secs after. 

Example: 

1. Only 1 pageview: 0-30 secs, we count as 30 secs
2. 1 pageviews + 1 ping: 30 sec + 10 secs = 40 secs
3. 1 pageviews + 10 ping: 30 secs + 10 secs * 10 = 130 secs

Sample query:

```sql
  # finding the timeonpage for each pageview event
SELECT
  DATE(collector_tstamp) AS Date,
  MIN(event_id) AS temp_pageview_id,
  MIN(domain_userid),
  MIN(domain_sessionid),
  MIN(page_urlpath),
  SUM(CASE
      WHEN event = 'page_ping' THEN 1
      ELSE 0 END) AS ping_count,
  SUM(CASE
      WHEN event = 'page_ping' THEN 1
      ELSE 0 END)*10 + 30 AS time_on_page
FROM
  `snowplow.snowplow.events`
WHERE
  DATE(collector_tstamp) = "2018-09-01"
  AND event IN ('page_view',
    'page_ping')
GROUP BY
  Date,
  domain_userid,
  domain_sessionid,
  page_urlpath;
```

### Time on Page by pagepath

Based on the query above, we can find the avg time on page for each pagepath, similar with the pagepath report from GA. 

```sql
  # avg time on page by pagepath
SELECT
  top.Date,
  top.pagepath,
  AVG(top.time_on_page)
FROM (
  SELECT
    DATE(collector_tstamp) AS Date,
    MIN(event_id) AS temp_pageview_id,
    MIN(domain_userid),
    MIN(domain_sessionid),
    MIN(page_urlpath) AS pagepath,
    SUM(CASE
        WHEN event = 'page_ping' THEN 1
        ELSE 0 END) AS ping_count,
    SUM(CASE
        WHEN event = 'page_ping' THEN 1
        ELSE 0 END)*10 + 30 AS time_on_page
  FROM
    `snowplow.snowplow.events`
  WHERE
    DATE(collector_tstamp) = "2018-09-01"
    AND event IN ('page_view',
      'page_ping')
  GROUP BY
    Date,
    domain_userid,
    domain_sessionid,
    page_urlpath) top
GROUP BY
  Date,
  top.pagepath;
```



## Finding Scroll depth

There are a few relevant columns from the events table for finding scroll depth and viewability: `pp_yoffset_max`, `pp_yoffset_min`, `doc_height`.

My guess: `doc_height` is the pixel count of page, `pp_yoffset_min` is the first pixel in the y-axis tracked in the beginning of event and `pp_yoffset_max` is the last pixel tracked. In general, we expect `pp_yoffset_min` <= `pp_yoffset_max` <= `doc_height`. To find scroll depth, we will need to find the maximum of `pp_yoffset_max` for all page_view or page_ping event, divided by the doc_height. To simplify, we ignore the case when multiple page view to the same pagepath.

Example: A pageview event with 1 page_view, followed by 2 page_ping events. let's say the article is 20000 pixel long. (should expect `doc_height` = 20000 for all 3 rows), and the MAX of `pp_yoffset_max` for all 3 rows are 15000. Then scroll depth = 15000/20000 * 100% = 75%.

Sample query:

```sql
SELECT
  DATE(collector_tstamp) AS Date,
  MIN(event_id) AS temp_pageview_id,
  MIN(domain_userid),
  MIN(domain_sessionid),
  MIN(page_urlpath) AS pagepath,
  MAX(pp_yoffset_max) AS max_depth,
  MAX(doc_height) AS doc_depth,
  MAX(pp_yoffset_max) / MAX(doc_height) AS scroll_depth
FROM
  `snowplow.snowplow.events`
WHERE
  DATE(collector_tstamp) = "2018-09-01"
  AND event IN ('page_view',
    'page_ping')
GROUP BY
  Date,
  domain_userid,
  domain_sessionid,
  page_urlpath;
```

ref: create a lookup table on pageview_id https://www.snowflake-analytics.com/blog/data-modeling/modelling-pageviews/

# Modelling sessions level metric

For modelling session level metric, we will need to link multiple pageview event for a single session. Similar with how GA define sessions: https://support.google.com/analytics/answer/2731565?hl=en

In Snowplow events table, we can use a column `domain_sessionid` 

```sql
SELECT
  DATE(collector_tstamp) AS Date,
  COUNT(DISTINCT domain_sessionid) AS Sessions
FROM
  `snowplow.snowplow.events`
WHERE
  DATE(collector_tstamp) >= "2018-08-01"
  AND event = 'page_view'
GROUP BY
  Date
ORDER BY
  Date;
```

For session duration, it would be a bit more tricky but we can apply similar strategy with Time On Page, and then sum up the time on page on multiple pageviews within a sinlge session. 

For more info about modelling sessions, please ref: https://www.snowflake-analytics.com/blog/data-modeling/modelling-sessions/




