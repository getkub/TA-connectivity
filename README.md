# TA-connectivity


The TA-connectivity app can be used to gather host connectivity information. It leverages the multiprocessing library in python using a pool of 4 workers. It contains 3 different scripts namely; ping.py, webping.py and connect.py.pyclbr

This app adds 3 modular inputs to any Splunk installation:
1. ping
2. webping
3. connect

Sample CSV:
```CSV
hostname,port,url
www.google.com,80,https://encrypted.google.com
www.yahoo.com,80,http://www.yahoo.com
4.2.2.2,53,http://www.twiter.com
8.8.8.8,53,http://www.linkedin.com

```


## Ping Modular Input
This input extracts hostnames or IP addreses from the host_field header column of the csv defined by the input.
If workers is not defined, it uses a default of 4 workers.

```
[ping:///path/to/lookup/file]
host_field = hostname
workers = 4
interval = 600
```

## webping modular input
The following input configuration would test web connectivity to all hosts in the csv using the URLs stored in the "host" column
The modular input uses a default of 4 workers when not configured, and a web timeout of 5s.

```
[webping:///path/to/lookup/csv]
host_field = hostname
workers = 4
web_timeout = 5
interval = 600
```


## connect modular input
This tests the connection to a host on a specified port.
If port field is not specified, it expects the host_field to be in the format *hostname:port* or *IP:port*.

*Example 1*
```
[connect:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt]
host_field = hostname
port_field = port
workers = 5
interval = [<number>|<cron schedule>]
```

*Example 2*
```
[connect:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile2.txt]
host_field = socket
workers = 4
interval = 600
disabled=true
````

*Sample CSV for Example 2*
```csv
socket
www.google.com:80
www.yahoo.com:80
4.2.2.2:53
8.8.8.8:53
```

# Testing

A number of disabled modular inputs come with the application which can be enabled.
These would write data to the default index.


The modular inputs can be tested with the following commands:

## Test 1: Testing the ping modular input above
```
/opt/splunk/bin/splunk cmd splunkd print-modinput-config ping ping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt | /opt/splunk/bin/splunk cmd python /opt/splunk/etc/apps/TA-connectivity/bin/ping.py
```

### Output of Test1:
```xml
<stream>
    <event stanza="ping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:03:54 BST ,action=ping failed,status=998,src=splunk,dst_hostname=www.google.com,dst_ip=216.58.210.36,description=,average_rtt=101.5,packet_loss=0%
        </data>
        <done />
    </event>
    <event stanza="ping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:03:54 BST ,action=ping failed,status=998,src=splunk,dst_hostname=www.yahoo.com,dst_ip=46.228.47.115,description=,average_rtt=105.5,packet_loss=0%
        </data>
        <done />
    </event>
    <event stanza="ping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:03:54 BST ,action=ping succeeded,status=200,src=splunk,dst_hostname=4.2.2.2,dst_ip=4.2.2.2,description="icmp_seq=1 ttl=58;icmp_seq=2 ttl=58",average_rtt=100.5,packet_loss=0%
        </data>
        <done />
    </event>
    <event stanza="ping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:03:54 BST ,action=ping succeeded,status=200,src=splunk,dst_hostname=8.8.8.8,dst_ip=8.8.8.8,description="icmp_seq=1 ttl=55;icmp_seq=2 ttl=55",average_rtt=102.5,packet_loss=0%
        </data>
        <done />
    </event>
</stream>
```

## Test 2: WebPing modular input
```
/opt/splunk/bin/splunk cmd splunkd print-modinput-config webping webping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt | /opt/splunk/bin/splunk cmd python /opt/splunk/etc/apps/TA-connectivity/bin/webping.py
```

### Output of command:
```xml
<stream>
    <event stanza="webping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 03:54:35 BST,action=successful,status=200,src=splunk,dst=encrypted.google.com,url="https://encrypted.google.com",description=online</data>
        <done />
    </event>
    <event stanza="webping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 03:54:35 BST,action=successful,status=200,src=splunk,dst=www.yahoo.com,url="http://www.yahoo.com",description=online</data>
        <done />
    </event>
    <event stanza="webping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 03:54:35 BST,action=successful,status=200,src=splunk,dst=www.twitter.com,url="http://www.twitter.com",description=online</data>
        <done />
    </event>
    <event stanza="webping:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 03:54:35 BST,action=successful,status=200,src=splunk,dst=www.linkedin.com,url="http://www.linkedin.com",description=online</data>
        <done />
    </event>
</stream>
```

## Test 3: Connect modular input
```
/opt/splunk/bin/splunk cmd splunkd print-modinput-config connect connect:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt | /opt/splunk/bin/splunk cmd python /opt/splunk/etc/apps/TA-connectivity/bin/connect.py
```

### Output of command:
```xml
<stream>
    <event stanza="connect:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:44:49 BST ,action=connection succeeded,status=200,src=splunk,dst_hostname=www.google.com,dst_ip=216.58.210.36,description=Connection successful to host=www.google.com on port=80
        </data>
        <done />
    </event>
    <event stanza="connect:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:44:49 BST ,action=connection succeeded,status=200,src=splunk,dst_hostname=www.yahoo.com,dst_ip=46.228.47.115,description=Connection successful to host=www.yahoo.com on port=80
        </data>
        <done />
    </event>
    <event stanza="connect:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:44:49 BST ,action=connection succeeded,status=200,src=splunk,dst_hostname=4.2.2.2,dst_ip=4.2.2.2,description=Connection successful to host=4.2.2.2 on port=53
        </data>
        <done />
    </event>
    <event stanza="connect:///opt/splunk/etc/apps/TA-connectivity/lookups/hostfile.txt" unbroken="1">
        <data>
            06/19/2016 04:44:49 BST ,action=connection succeeded,status=200,src=splunk,dst_hostname=8.8.8.8,dst_ip=8.8.8.8,description=Connection successful to host=8.8.8.8 on port=53
        </data>
        <done />
    </event>
</stream>
```
