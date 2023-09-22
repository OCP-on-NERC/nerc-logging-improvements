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
date and node and stored in archivel storage until requested. If this is 
the case then storing the logs in a compressed lossless format could allow 
them fairly grainular control over which logs they are retrieving, which 
would could reduce CPU time on the stored machine. This does have some
pitfalls as the researchers would be getting the whole of the logs on that
node for that timeframe. So if the researchers wanted/needed control over
which log levels they were looking at, that would need to be parced between
the decompression and presentation of the logs. The downside to this is it 
could increase the amount of requests being made to the server if they arnt
getting the exact logs they thought they were looking for.
