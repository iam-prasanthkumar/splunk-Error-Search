# splunk-Error-Search
1) Search the Error
index=app_logs "OutOfMemoryError"

2)With Time, Host, Service
index=app_logs "OutOfMemoryError"
| table _time host source sourcetype _raw

3) Count by Hosts
index=app_logs "OutOfMemoryError"
| stats count by host
| sort - count

===================Connection Timeout (Database / Network)

4) Search Both Error At Time
index=app_logs ("Connection timeout" OR "SQLTimeoutException")

5) Service & Thread-Level Analysis
index=app_logs ("Connection timeout" OR "SQLTimeoutException")
| rex "com\.app\.(?<service>[^ ]+)"
| stats count by service

6) Time Trend (Spike Detection)
index=app_logs ("Connection timeout" OR "SQLTimeoutException")
| timechart span=5m Count

====================NullPointerException 

7) index=app_logs ("NullPointerException")


=====================Error Type Classification===================
8) index=app_logs ("Connection timeout" OR "SQLTimeoutException" OR "NullPointerException" OR "OutOfMemoryError" )
| eval error_type=case(
     like(_raw,"%OutOfMemoryError%"), "OutOfMemoryError", 
     like(_raw,"%Connection timeout%") OR Like (_raw,"%SQLTimeoutException%"), "Connection timeout",
      like(_raw,"%NullPointerException%"), "NullPointerException", 
)
| stats count by error_type

====================Production Support Live Monitoring======================
9) index=app_logs ("Connection timeout" OR "SQLTimeoutException" OR "NullPointerException" OR "OutOfMemoryError")
earliest=-15m
| table _time host source _raw

==================================Alert Queries (Real-time use)==================
a) OutOfMemoryError alert

inder=app_logs "OutOfMemoryError"
| stats count
| where count>0

B) DB Timeout Threshold Alert
index=app_logs ("Connection timeout" OR "SQLTimeoutException")
| stats count
| where count>5

===============================Find slow API responses

10) index=app_logs
| where response_time > 3000
| table _time host service endpoint response_time
| sort - response_time


