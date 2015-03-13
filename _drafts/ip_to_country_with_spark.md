
1. How to translate from IP to Country
2. Why existing libraries are not compatible with Spark
3. Reinvent the translator

  3.1. Reading the database
  3.2. How to look up for a particular IP for its IP range
  3.3. Binary search algorithm
  3.3.1. Those ranges are not overlapped
    Checking it

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
  3.3.2. The algorithm
  3.3.3. Implementaion
