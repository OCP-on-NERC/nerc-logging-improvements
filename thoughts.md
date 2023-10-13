# Unfiltered thoughts for logging improvements

Per Heidi, the researchers are not allowed to download the data from the 
MOC. As the researchers are already familiar with Loki and Grafana, my 
instinct would be to move the logs into a seperate cluster Loki instance 
that the researchers would have access to. However there are some 
concerns with this as the researchers would have the unfiltered view of 
the logs leading to privacy concerns. This may be able to be addressed by
filtering censoring the logs before transit or configuring the 
researchers permissions with RBAC on the 'research cluster'. I believe for
RBAC to work here we may need to assign it on a per node topic level, but
this is likely more trouble than its worth and would likely require
ongoing maintaince. Heidi also mentioned that the researchers may prefer
.csv files, however this could still lead to privacy concerns. If the
researchers have access to the files, they would likely be able to copy
and paste the data and there may be concerns for ways to exfiltrate the
files as a whole. However, having the data is a .csv format could make it
much easier for the researchers to perform ML or hand written algorithms
for more in depth data analysis then they might get through a GUI in say
Grafana.

Depending on what the team descides is the best method for the researchers
and data privacy concerns, will likely impact the decision of how the logs
are to be stored and transfered. If for example, the team descides to go 
with .csv files, the logs could be aggrigated into individual files by
date and node and stored in archival storage until requested. If this is 
the case then storing the logs in a compressed lossless format could allow 
them fairly grainular control over which logs they are retrieving, which 
would could reduce CPU time on the stored machine. This does have some
pitfalls as the researchers would be getting the whole of the logs on that
node for that timeframe. So if the researchers wanted/needed control over
which log levels they were looking at, that would need to be parced between
the decompression and presentation of the logs. The downside to this is it 
could increase the amount of requests being made to the server if they arnt
getting the exact logs they thought they were looking for.

Space requirements for archival storage: based on the current usage of the 
cluster, as addressed by [computate and larsks](https://github.com/nerc-project/operations/issues/240)
we will need about 50TB of CEPH storage per year at the current usage rates.
The prod system shouldn't need any more than it currently has as storage 
should be cleared once we can confirm the receipt of the archives.

At this time I believe setting up a cronjob on the cluster to move the logs
over will be an effective solution. Depending on the needs of the
researchers, the logs could be moved over hourly, daily, or weekly. It
seems to me that more frequently would be the better of the two unless we
have some reason we can't do that for CPU or networking constraints. Doing
more frequent 'backups' will reduce the size of the package over the wire,
and help prevent any issues.

It's my understanding that the the AWS cli tool and MC cli tool could both
be used for this. The issues the we are currently experienceing with them,
is likely do to the size of the transfers causing a timeout of the network
connection. Shorter, more frequent updates should mitigate this risk.
However, that doesn't solve the problem of transfering over the data 
currently stored on the cluster. One not great solution would be to just
ignore that data and start transfering the current logs. Although the 
researchers would lose the data that has already been aggrigated. On the 
other hand, they have already lost all the data that has been dumped over
time anyways.

uses should be able to access the last month of their logs. and the archived logs. the user dont have access to the vpn/firewall so how can they retreive those logs?

are infra logs just openshift-[namespace]?
do they need just their pod logs?
do they need all openshift infra logs?
what do admin need access to?

