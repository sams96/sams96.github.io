---
title: Always Sort by ID — An Interesting Gotcha
url: /always-sort-by-ID
date: 2025-05-27
categories:
  - programming
#tootLink: "https://toot.io/@mondoman712/114071633084030867"
---

If you're writing a database query for your web service and there's pagination,
you should always sort by a unique field like ID. This should also be done on
top of any other sorting. Otherwise you can get double entries or lose results
across pages.<!--more--></br>

A sort can be non-determinstic, as in if you have multiple items with the same
value in the field you are sorting by, there are multiple valid ways to sort it.
This can be okay, until you need that sorting to persist through multiple
queries, as you will when using pagination. This means that your query for your
second page could sort your results differently and give the same entry again,
or completely miss an entry. In the example below, both tables show valid ways
to sort by Customer Name, but give different results which in this case leads to
one order being missed and another shown twice.

### Page 1

The first two results are returned.

| | **ID** | **Customer name** | **Item Ordered** |
| --- | --- | --- | --- |
| -> | 1 | Alan | Apple |
| -> | 2 | Betty | Banana |
|  | 3 | Betty | Grape |
|  | 4 | Callum | Apple |

### Page 2

Now with an offset of 2 the third and forth results are returned, but because we
are sorting in an non-determinstic way, the order has changed and we get
different results than expected.

| | **ID** | **Customer name** | **Item Ordered** |
| --- | --- | --- | --- |
| | 1 | Alan | Apple |
| | 3 | Betty | Grape |
| -> | 2 | Betty | Banana |
| -> | 4 | Callum | Apple |

This only applies when using the classic `LIMIT` and `OFFSET` method of
pagination. [Other methods are
available](https://www.citusdata.com/blog/2016/03/30/five-ways-to-paginate/),
but those tend to come with other downsides that make them less viable for the
type of web services I work on, requiring either more resources or less
flexibility.

The simple solution here is to just always include a sort by a unique ID, on top
of any existing sorting or any requested by the client. That way your sorting is
always deterministic and no results are lost.
