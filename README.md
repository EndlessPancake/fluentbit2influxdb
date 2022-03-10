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
 SELECT host, COUNT(*) FROM STREAM:nginx WINDOW TUMBLING (30 SECOND) GROUP BY host;
```

# My Problem
Q. When nginx has nothing to output to logs (no requests), fluent-bit "Stream Processing" do nothing(no count).
   In this case, I wanted to save "0" to influxdb by fluent-bit. What should I do in such a case?
### current
```
> select * from nginx2 WHERE time >= '2022-03-10T08:25:35.713188355Z'
name: nginx2
time                           Counts        remote_addr
----                           ------------- -----------
2022-03-10T08:25:35.713188355Z 91            34.97.xxx.xxx
2022-03-10T08:26:35.713183227Z 91            34.97.xxx.xxx
2022-03-10T08:27:05.71316811Z  91            34.97.xxx.xxx
2022-03-10T08:28:05.713163737Z 91            34.97.xxx.xxx
2022-03-10T08:31:05.713161209Z 91            128  34.97.244.79
```
### wish
```
> select * from nginx2 WHERE time >= '2022-03-10T08:25:35.713188355Z'
name: nginx2
time                           Counts        remote_addr
----                           ------------- -----------
2022-03-10T08:25:35.713188355Z 91            34.97.xxx.xxx
<30 SECONDS later>             0             xxx.xxx.xxx.xxx    // <-- count "0"
2022-03-10T08:26:35.713183227Z 91            34.97.xxx.xxx
2022-03-10T08:27:05.71316811Z  91            34.97.xxx.xxx
<30 SECONDS later>             0             xxx.xxx.xxx.xxx    // <-- count "0"
2022-03-10T08:28:05.713163737Z 91            34.97.xxx.xxx
<30 SECONDS later>             0             xxx.xxx.xxx.xxx    // <-- count "0"
2022-03-10T08:29:35.713167177Z 91            34.97.xxx.xxx
<30 SECONDS later>             0             xxx.xxx.xxx.xxx    // <-- count "0"
<30 SECONDS later>             0             xxx.xxx.xxx.xxx    // <-- count "0"
<30 SECONDS later>             0             xxx.xxx.xxx.xxx    // <-- count "0"
2022-03-10T08:31:05.713161209Z 91            128  34.97.244.79
```
