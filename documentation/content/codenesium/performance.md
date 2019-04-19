+++
title = "Performance"
description = ""
weight = 1
alwaysopen = true
+++
With our architecture it's easy to achieve > 5000 requests per second with 10 year old hardware. 

This test was ran from localhost on the server. It returns a single address record.
Comparable performance was found running from another machine. 
The server was CPU bound by dotnet.exe while running this example.


```
Windows Server 2012
SQL Server 2014
8GB ram.
12 Cores CPU X5650 @ 2.67GHz 
AdventureWorks2012 database from https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks
Tested with bombardier from https://github.com/codesenberg/bombardier

C:\Users\Administrator\Downloads>bombardier http://localhost/testSite/api/addres
ses/1 --header="api-version:1.0" --duration=60s --connections=125
Bombarding http://localhost/testSite/api/addresses/1 for 1m0s using 125 connecti
ons
[========================================================================] 1m0s
Done!
Statistics        Avg      Stdev        Max
  Reqs/sec      5129.00    1963.66   76022.81
  Latency       24.48ms     3.41ms   241.01ms
  HTTP codes:
    1xx - 0, 2xx - 306391, 3xx - 0, 4xx - 0, 5xx - 0
    others - 0
  Throughput:     2.70MB/s
```


```
Windows Server 2012
SQL Server 2014
8GB ram.
2 Cores CPU X5650 @ 2.67GHz 
AdventureWorks2012 database from https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks
Tested with bombardier from https://github.com/codesenberg/bombardier

C:\Users\Administrator\Downloads>bombardier http://localhost/testSite/api/addres
ses/1 --header="api-version:1.0" --duration=60s --connections=125
Bombarding http://localhost/testSite/api/addresses/1 for 1m0s using 125 connecti
ons
[========================================================================] 1m0s
Done!
Statistics        Avg      Stdev        Max
  Reqs/sec       970.43    1138.21   19506.83
  Latency      132.26ms    87.41ms      3.14s
  HTTP codes:
    1xx - 0, 2xx - 56720, 3xx - 0, 4xx - 0, 5xx - 0
    others - 0
  Throughput:   512.57KB/s
```