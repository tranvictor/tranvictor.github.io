---
layout: post
title: "Geographic report with Apache Spark from Nginx access log, experience from a newbie"
date: 2015-03-13
image: /images/spark_experiment.png
tags: python spark big-data
---

There are many blog posts and articles around the web saying that Spark is blazing fast because of in-memory distributed processing. Well, I would see it myself so I did some experiments on it.

*Note: This might be a long post, I'm sorry for not bringing you straight to the point. I just want to write so in couple of months later I can remember what I have done.*

## Experiment description

In this experiment, I will try to do a very well-known log mining task, which is generating geographic report from Nginx access log.

Environment specifications:

  1. 2.3 GHz Intel Core i7 (4 process cores)
  2. 16 GB 1600 MHz DDR3
  3. 50 MB access log file

## How do we do this experiment

Because Apache Spark supports API in Java, Scala and Python so I chose Python due to its readability.

  1. Write a Python program to see how long does it take to generate the report
  2. Using Apache Spark on that program to see many time faster it is in compare to the first one.

## Here we go

Now it's time for us to start doing the experiment.

### How do we generate the report

It's pretty straight forward to do this task, we can just read the log line by line, determine what the country does that request come from, increase that country counter. The result would be a dictionay of country name as keys and number of requests as values. I think it's pretty much like this:

```python
  {
    'Canada': 123,
    'Unknown': 123,
    'Lithuania': 123,
    'Argentina': 123,
    'Bahrain': 123,
    'Saudi Arabia': 123,
    ...
  }
```

### Regular program

I came up to use `python-geoip` package to translate IP to country name.

You can install the package using pip like this:

```
pip install python-geoip
pip install python-geoip-geolite2
```
This is my regular python program.

```python
import datetime
from geoip import geolite2

begining = datetime.datetime.now()
if __name__ == "__main__":
    ips = {}
    for line in open('access.log'):
        # getting ip from log line, it's the first non-space string
        ip = line.split(' ')[0]
        match = geolite2.lookup(ip)
        if match is not None:
            ips[match.country] = (ips.get(match.country) or 0) + 1
        else:
            ips["Unknown"] = (ips.get("Unknown") or 0) + 1
    ending = datetime.datetime.now()
    print ips
    print "processed in: %s" % (ending - begining)
```

After running this script, we got:

```
{'BD': 695, 'BE': 18171, 'BG': 139, 'BA': 27, 'BL': 21, 'Unknown': 4723, 'BN': 2, 'JP': 2140, 'BI': 1, 'JM': 6, 'JO': 5, 'BQ': 71, 'BR': 1569, 'BY': 67, 'RU': 487, 'RS': 88, 'RO': 817, 'GT': 3, 'GR': 64, 'BH': 1, 'GY': 30, 'GE': 4, 'GB': 13164, 'GL': 1, 'GI': 19, 'GH': 2, 'OM': 4, 'TN': 1, 'BW': 22, 'HR': 4, 'HT': 4, 'HU': 65, 'HK': 112, 'HN': 1, 'PS': 65, 'PT': 267, 'PY': 23, 'PA': 23, 'PE': 6, 'PK': 810, 'PH': 498, 'PL': 406, 'ZM': 54, 'EE': 61, 'EG': 111, 'ZA': 29501, 'EC': 97, 'IT': 579, 'AO': 16, 'ZW': 57, 'SA': 87, 'ES': 355, 'MD': 52, 'MA': 54, 'MO': 31, 'MK': 9, 'UA': 817, 'MX': 130, 'IL': 361, 'FR': 3720, 'FI': 58, 'FJ': 2, 'NL': 6867, 'NO': 494, 'NA': 44, 'NG': 3, 'NZ': 2789, 'CH': 590, 'CO': 152, 'CN': 3713, 'CL': 63, 'CA': 14187, 'CD': 2, 'CZ': 175, 'CR': 48, 'CW': 1227, 'KE': 5, 'SR': 1, 'SK': 24, 'KR': 86, 'SI': 31, 'SN': 21, 'SC': 51, 'KZ': 3, None: 1169, 'SG': 616, 'SE': 1366, 'DO': 2, 'DK': 923, 'DE': 5865, 'DZ': 56, 'US': 49443, 'LB': 1, 'TW': 87, 'TT': 2, 'TR': 153, 'LK': 50, 'LI': 1, 'LV': 62, 'LT': 70, 'LU': 30, 'TH': 270, 'AE': 41, 'VE': 53, 'IQ': 45, 'IS': 2, 'IR': 21, 'AM': 3, 'AL': 5, 'VN': 20978, 'AR': 46, 'AU': 2232, 'AT': 337, 'IN': 9469, 'IE': 73, 'ID': 329, 'MY': 8, 'QA': 1}
processed in: 0:00:58.259646
```

As you can see, it took about 58s to analyze 50 MB log file. Hmm, pretty slow but whatever, using Spark would make it blazing fast I thought.

### The program using Apache Spark

The code:

```python
import sys

from pyspark import SparkContext

def get_country_from_line(line):
    try:
        from geoip import geolite2
        ip = line.split(' ')[0]
        match = geolite2.lookup(ip)
        if match is not None:
            return match.country
        else:
            return "Unknown"
    except IndexError:
        return "Error"

if __name__ == "__main__":
    sc = SparkContext(appName="PythonAccessLogAnalyzer")
    rdd = sc.textFile("/Users/victor/access.log").map(get_country_from_line)
    ips = rdd.countByValue()

    print ips
    sc.stop()
```
You will find it weird when I import geolite2 inside a function but it is neccessary to make this code able to run in Spark because Spark will ship the method `get_country_from_line` to its workers, that wouldn't know what `geolite2` is. Weird right?

The result is:

```
Job 0 finished: countByValue at /Users/victor/Downloads/spark-1.2.1-bin-hadoop2.4/examples/src/main/python/access_log_analyzer.py:20,
took 33.927496 s
defaultdict(<type 'int'>, {'BD': 695, 'BE': 18171, 'BG': 139, 'BA': 27, 'BL': 21, 'Unknown': 4723, 'BN': 2, 'JP': 2140, 'BI': 1, 'JM': 6, 'JO': 5, 'BQ': 71, 'BR': 1569, 'BY': 67, 'RU': 487, 'RS': 88, 'RO': 817, 'GT': 3, 'GR': 64, 'BH': 1, 'GY': 30, 'GE': 4, 'GB': 13164, 'GL': 1, 'GI': 19, 'GH': 2, 'OM': 4, 'BW': 22, None: 1169, 'HR': 4, 'HT': 4, 'HU': 65, 'HK': 112, 'HN': 1, 'PS': 65, 'PT': 267, 'PY': 23, 'PA': 23, 'PE': 6, 'PK': 810, 'PH': 498, 'PL': 406, 'ZM': 54, 'EE': 61, 'EG': 111, 'ZA': 29501, 'EC': 97, 'AL': 5, 'VN': 20978, 'ZW': 57, 'ES': 355, 'MD': 52, 'MA': 54, 'MO': 31, 'US': 49443, 'UA': 817, 'MX': 130, 'IL': 361, 'FR': 3720, 'FI': 58, 'FJ': 2, 'NL': 6867, 'NO': 494, 'NA': 44, 'NG': 3, 'NZ': 2789, 'CH': 590, 'CO': 152, 'CN': 3713, 'CL': 63, 'CA': 14187, 'CD': 2, 'CZ': 175, 'CR': 48, 'CW': 1227, 'KE': 5, 'SR': 1, 'SK': 24, 'KR': 86, 'SI': 31, 'SN': 21, 'SC': 51, 'KZ': 3, 'SA': 87, 'SG': 616, 'SE': 1366, 'DO': 2, 'DK': 923, 'DE': 5865, 'AT': 337, 'DZ': 56, 'MK': 9, 'LB': 1, 'TW': 87, 'TT': 2, 'TR': 153, 'LK': 50, 'LI': 1, 'TN': 1, 'LT': 70, 'LU': 30, 'TH': 270, 'AE': 41, 'VE': 53, 'IQ': 45, 'IS': 2, 'IR': 21, 'AM': 3, 'IT': 579, 'AO': 16, 'AR': 46, 'AU': 2232, 'LV': 62, 'IN': 9469, 'IE': 73, 'ID': 329, 'MY': 8, 'QA': 1})
```

Wow wow wow, WTF, 33.9s. Not impressed at all, how slow!

### What is the problem?

The problem is that I was right that I felt weird of importing `geolite2` inside a function. That method is not just spread to its work, it will be used whenever the program process a log line. Thus we ran this importing stupidity too many time which led to slow down the program significantly.

### How do we fix this

Spark provides a way to broadcasting variables to its workers so workers don't need to reload that variables. I tried to broadcast the `geolite2`, however, it doesn't work because `geolite2` is a package, which references to thread and complex things internally so Spark can't handle that complex box.

If you find a way to broadcast `geolite2` or making the program faster and still using `geolite2`, please leave a comment. I really appreciate it.

Eventually I thought to fix this, I would have to implement my own IP to Country translator which can be broadcasted using Spark.

### Fix and Improve the Spark version

I came up with this code:

```python
import sys
import csv
import bisect

from pyspark import SparkContext

ranges = []
keys = []

with open('/Users/victor/Downloads/GeoIPCountryWhois.csv', 'rb') as f:
  reader = csv.reader(f)
  for row in reader:
      ip_start = int(row[2])
      ip_end = int(row[3])
      ranges.append((ip_start, ip_end, row[5], row[0], row[1]))
      keys.append(ip_start)

def lookup(ip, r, k):
    (o1, o2, o3, o4) = ip.split('.')
    i_ip = 16777216 * int(o1) + 65536 * int(o2) + 256 * int(o3) + int(o4)

    index = bisect.bisect(k, i_ip)
    result = None
    if index > 0:
        index -= 1
        found_range = r[index]

        if found_range[1] > i_ip:
            result = found_range[2]
        else:
            result = "Unknown"
    else:
        result = "Unknown"
    return result

if __name__ == "__main__":
    sc = SparkContext(appName="PythonAccessLogAnalyzer")
    ip_to_country_db = sc.broadcast(ranges)
    ip_to_country_keys = sc.broadcast(keys)
    def get_country_from_line(line):
        try:
            ip = line.split(' ')[0]
            match = lookup(ip, ip_to_country_db.value, ip_to_country_keys.value)
            return match
        except IndexError:
            return "Error"

    rdd = sc.textFile("/Users/victor/access.log").map(get_country_from_line)
    ips = rdd.countByValue()

    print ips
    sc.stop()
```

You might get confused about this code but I'm not gonna explain this because the post is too long now. I will write a different post to explain this code which contains some little tricks on algorithm that you might interest.

And now, the result is:

```
Job 0 finished: countByValue at /Users/victor/Downloads/spark-1.2.1-bin-hadoop2.4/examples/src/main/python/access_log_analyzer_with_custom_translator.py:49,
took 2.267526 s
```

Hah, bingo, 2.26s. Pretty fast.

## Bigger data

I ran this improved version with a bigger file, 5 GB, it took about 119.2s. The previous version took over an hour. The regular program took forever, I couldn't wait for it to finish the task.

## Conclusion

Spark helps to solve this kind of problem seriously faster with parallelize the process. However we have to be awared of variable shipping to make it work efficiently, otherwise it will still be slow.

  1. Regular program needs 58s to analyze 50 MB log file.
  2. Bad Spark version needs 33s to analyze 50 MB log file.
  3. Good Spark version needs 2.26s to analyze 50 MB log file.
  4. Bad Spark version needs an hour to analyze 5 GB log file.
  5. Good Spark version needs 119.2s to analyze 5 GB log file.

*PLEASE FEEL FREE TO LEAVE A COMMENT.*
