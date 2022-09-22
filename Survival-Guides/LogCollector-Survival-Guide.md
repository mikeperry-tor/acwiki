## LogCollector Survival Guide

Log collector is running at `probetelemetry-01.torproject.org`.

### Quick Commands

Switch to torprobe user:
```
sudo -u torprobe bash
```

### Components

#### Log Collector
[Log Collector](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/logcollector), running as [torprobe.service](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/logcollector-admin/-/blob/main/LogCollector/Service/torprobe.service), receives data from vantage point and store them into the disk.

#### V2Ray
[V2Ray](https://github.com/v2fly/v2ray-core), running as [v2ray.service](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/logcollector-admin/-/blob/main/V2Ray/Service/v2ray.service), prevent vantage point from connecting to the log collector directly to prevent revaluing vantage point's identity.

### TestingTargetUpdater
[TestingTargetUpdater](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/logcollector-admin/-/blob/main/TestingTargetUpdater/Script/update_bridge.sh), running as [torprobed-update-testing-target.service ](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/logcollector-admin/-/blob/main/TestingTargetUpdater/Service/torprobed-update-testing-target.service), update testing target to be distributed to vantage points from [(Restricted) bridgelines repo](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/bridgelines).

### Report Generator

[Report Generator](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/logcollector-admin/-/blob/main/ReportGenerator/Script/processorScript.sh), running as [torprobed-publish-result.service](https://gitlab.torproject.org/tpo/anti-censorship/connectivity-measurement/logcollector-admin/-/blob/main/ReportGenerator/Service/torprobed-publish-result.service), generate report from log collector's data.