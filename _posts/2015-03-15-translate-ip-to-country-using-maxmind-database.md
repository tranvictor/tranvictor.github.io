---
layout: post
title: "Translate IP to Country using Maxmind database (reinvent the wheel)"
date: 2015-03-15
image: /images/myself.jpg
tags: python algorithm
---

This blog post is explanation for the algorithm I used in [previous post](/2015/03/13/geographic-report-with-apache-spark-from-nginx-access-log).

The algorithm is to determine which country does an IP come from.

*Input: an IPv4 string (eg. 199.191.56.130)*

*Output: country name (eg. United States)*

## Existing libraries

  * [pygeoip](https://github.com/appliedsec/pygeoip)
  * [python-geoip](https://github.com/mitsuhiko/python-geoip)
  * [geoip2](https://github.com/maxmind/GeoIP2-python)
  * ...

## Why did I have to "reinvent the wheel"

They are all good for translating IP to Country. They also provides easy-to-use APIs, however I didn't find a way to use them with Apache Spark. I had to import them whenever I want to process something (eg. a line from a log file) which drains too much performance for library initialization.

Because of that reason, I needed to write a piece of code to translate IP to Country myself which should be simple, fast and be able broadcasted by Apache Spark.

## Mindmax Database

There's no straight algorithm to determine which country an IP comes from, I have to use a database that contains a map from an IP range to a country to know which range the IP belongs thus I know its country.

Mindmax provides simple, accurate and frequently updated that kind of database. They have `GeoIP2` and `GeoLagacy` in binary and CSV format. I used GeoIP2 CSV version because of its simplicity and readabily.

### Download the database

You can download it from [here](http://dev.maxmind.com/geoip/legacy/geolite/), please choose GeoLite Country.

### Database format

In this database, there's only one CSV file containing many lines. Each line is a mapping from IP range to country and it looks like this:

```
"2.17.252.0","2.17.255.255","34733056","34734079","DE","Germany"
```

It starts with two IPs defining the IP range follow with 2 numbers that are the previous 2 IPs in integer. It ends with the country code and country name. So that line means every IPs from *2.17.252.0* to *2.17.255.255* comes from *Germany*. It also means every IPs in integer between *34733056* and *34734079* is *Germany*.

Working with integer is always simpler and faster than string so I decided to use IP in integer form so I used only:

```
"34733056" "34734079" "Germany"
```

to define an IP range.

## Our problem

Given a specific IP, our problem now turns into finding a range which the IP belongs to. So if "2.17.252.10" is our IP, the algorithm should know it belongs to `["2.17.252.0", "2.17.255.255"]` so it comes from "Germany".

## The algorithm

Because I decided to use integer to represent IP address so I need to find a way to convert from IP address to integer. It's pretty simple to figure out the formular like this:

```
Integer for o1.o2.o3.o4: 16777216 * o1 + 65536 * o2 + 256 * o3 + o4
```

Next, I have to determine which range does that integer *x* belong to.

It's quite simple that I can loop through every given ranges *[a, b]*. If *a <= x <= b*, that range is the result and I know the corresponding country name. This algorithm is simple but slow because its lookup complexity is *O(n)* where n is number of given ranges (about 100k in the database).

It's just too slow for me, I looked at the database and thought those ranges shouldn't be overlapped. I checked to see if i was right or not by this peice of code:

```python
>>> ranges = []
>>> with open('GeoIPCountryWhois.csv', 'rb') as file:
...     reader = csv.reader(file)
...     for row in reader:
...         ranges.append((int(row[2]), int(row[3]), row[5]))
...
>>> ranges.sort(key=lambda e: e[0])
>>> overlapped = False
>>> for i in range(0, len(ranges)):
...     if i > 0 and ranges[i-1][1] > ranges[i][0]:
...         overlapped = True
...         break
...
>>> overlapped
False
```

I was right, that means if I sort those ranges *[ai, bi]* in respect of *ai* (*i* comes from *1..n*) I have

```
[a1 < b1] < [a2 < b2] < [a3 < b3] < ... < [an < bn].
```

If I found a range, there's no other ranges that the IP belongs to.

**Having *[ai, bi]* where *ai* is the greatest number which smaller or equal than *x* among *a1, a2, ..., an*, If *x <= bi*, *x* belongs to that range otherwise *x* doesn't belong to any ranges.**

To find the greatest number which is smaller or equal than *x*, I can use binary search which is way faster than previous algorithm with *O(logn)* complexity.

Python provides *bisect* library with binary searching algorithm built in so I didn't have to implement it myself.

## Implementation

This is my code using the algorithm above:

```python
def ip_to_country(ip):
  # collection of ranges, each range is a tupple of (starting ip, ending ip, country)
  ranges = []
  # array of starting ip for searching
  keys = []

  # read from database to collect ip ranges
  with open('GeoIPCountryWhois.csv', 'rb') as f:
      reader = csv.reader(f)
      for row in reader:
          ip_start = int(row[2])
          ip_end = int(row[3])
          ranges.append((ip_start, ip_end, row[5], row[0], row[1]))
          keys.append(ip_start)

  # convert IP to integer
  (o1, o2, o3, o4) = ip.split('.')
  i_ip = 16777216 * int(o1) + 65536 * int(o2) + 256 * int(o3) + int(o4)

  result = None

  # I dont have to sort the ranges because those ranges are already sorted in the
  # Mindmax database

  # using bisect to find the position i that every key goes before i will smaller
  # or equal to i_ip
  index = bisect.bisect(keys, i_ip)

  # if the position is greater than 0, the one before that position is the greatest
  # which is smaller or equal to i_ip
  if index > 0:

      # get that range
      found_range = ranges[index - 1]

      # if ending ip is greater or equal to i_ip, return the country name
      if found_range[1] >= i_ip:
          result = found_range[2]

      # otherwise, the IP doesn't belong to any ranges
      else:
          result = "Unknown"
  # if the position is 0, the IP doesn't belong to any ranges
  else:
      result = "Unknown"

  return result
```

## Conclusion

With this implementation, it's simple, fast and easy to understand. Most important thing is that it can be used effectively with Apache Spark by broadcasting the `ranges` and `keys` arrays.

I used this implementation with Apache Spark to process 5 GB log file in **110s**. I took an hour to process the same file using existing libraries.
