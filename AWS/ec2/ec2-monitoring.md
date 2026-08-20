1. Basic vs Detailed Monitoring
Every EC2 instance automatically sends certain metrics to Amazon CloudWatch.
You do not need to install the CloudWatch Agent for these default EC2 metrics.

| Basic | Detailed |
|---|---|
| Default EC2 monitoring | Optional |
| Most metrics every 5 min | Most metrics every 1 min |
| Lower granularity | Higher granularity |
| No additional EC2 monitoring charge | Additional charge |
| Fine for less critical workloads | Better for production/faster alarms |


Default CloudWatch metrics

CPUUtilization — percentage of CPU being used.
NetworkIn — bytes received by the instance.
NetworkOut — bytes sent by the instance.
NetworkPacketsIn — number of packets received.
NetworkPacketsOut — number of packets sent.
DiskReadOps — read operations on instance-store volumes.
DiskWriteOps — write operations on instance-store volumes.
DiskReadBytes — bytes read from instance-store volumes.
DiskWriteBytes — bytes written to instance-store volumes.
MetadataNoToken — successful IMDSv1 calls without a token.
MetadataNoTokenRejected — attempted IMDSv1 calls that were rejected because IMDSv1 is disabled


InstAall cloud watch agent

configure credentials 

configure file needs to be configured

| Term | Meaning | Our example |
|---|---|---|
| **Namespace** | Group/folder | `OnPrem/Ubuntu` |
| **Metric** | What you're measuring | `mem_used_percent` |
| **Dimension** | Which resource/entity | `Server=LAPTOP-HOO65F3F` |
| **Dimension Name** | Dimension key | `Server` |
| **Dimension Value** | Dimension's value | `LAPTOP-HOO65F3F` |
| **Datapoint Value** | Actual measurement | `31.5` |
| **Timestamp** | When measured | `00:52` |
| **Unit** | Measurement unit | `%` |
| **Statistic** | How values are summarized | `Average` |
| **Period** | Time bucket | `60 seconds` |'