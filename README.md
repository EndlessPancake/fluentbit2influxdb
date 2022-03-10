# fluentbit2influxdb
Stream Processing by fluent-bit 
## in_tail , out_influxdb
- td-agent-bit.conf
```
[INPUT]
    Name        tail
    Alias       nginx
    Path        /var/log/nginx/access.log
    
    # nginx is defined parsers.conf
    Parser      nginx

[OUTPUT]
#    name  stdout
#    match *
    Name          influxdb
    Match         *
    Host          127.0.0.1
    Port          8086
    Database      fluentbit
    Sequence_Tag  _seq

```
## Stream Processing
- stream_processing.conf
```
 SELECT host, COUNT(*) FROM STREAM:nginx WINDOW TUMBLING (5 SECOND) GROUP BY host;
```

# My Problem
Q. When nginx has nothing to output to logs (no requests), fluent-bit "Stream Processing" do nothing(no count).
   In this case, I wanted to save "0" to influxdb by fluent-bit. What should I do in such a case?
