---
title: Cheatsheet
date: 2023-12-30
draft: false
---

I think most people hate writing bash scripts. Bash took the concept of syntactic sugar and thought, "what if syntactic pain"?

So anyways, this is a collection of tricks for making sure your thing stays running in a non-forkbomb sort of way and some other stuff too.

Persistence/application check
```bash
if ! pgrep -f "myth.py" >/dev/null; then  
  setsid --fork python3 /var/atlassian/application-data/bitbucket/bin/myth.py </dev/null &>/dev/null &
fi
```

Restart autossh if one or both connections are down
```bash
#!/bin/bash

autossh_processes=$(pgrep -f autossh) 
process_count=$(echo "$autossh_processes" | wc -w)
if [ "$process_count" -ne 2 ]; then
  killall autossh
  /home/kali/start_autossh.sh
fi
```

Restart script on reboot
```
@reboot /home/kali/keep_autossh.sh >> /home/kali/klog.txt 2>&1
* * * * * /home/kali/keep_autossh.sh >> /home/kali/kmin_log.txt 2>&1
```

Send slack webhook on state change
```bash
#!/bin/bash

# File to store the previous connection state
STATE_FILE="/path/to/connection-state.txt"

# Run netstat -planet and check for "2222"
if netstat -planet | grep -q ":2222"; then
    # Port 2222 is open
    current_state="up"
else
    # Port 2222 is not open
    current_state="down"
fi

# Read the previous state from the state file
previous_state=$(cat "$STATE_FILE" 2>/dev/null)

# Check if the current state is different from the previous state
if [ "$current_state" != "$previous_state" ]; then
    # State has changed, send a notification
    echo "Connection state has changed from $previous_state to $current_state"
    
    # Replace this line with the action you want to take
    curl -X POST -d '{"message": "Connection state has changed from '"$previous_state"' to '"$current_state"'"}' "https://hooks.slack.com/workflows/REPLACE_ME"

    # Update the state file with the current state
    echo "$current_state" > "$STATE_FILE"
else
    # State has not changed, no need to send a notification
    echo "Connection state is still $current_state"
fi

```


https://www.mitiga.io/blog/mitiga-security-advisory-abusing-the-ssm-agent-as-a-remote-access-trojan

script bettercap
```bash
sudo bettercap -script /path/to/script.js
```

Start bettercap on specific channel capturing handshakes
```js
run('wifi.recon.channel 1');
run('wifi.recon on');
```

Start tcpdump on eth0 with no name resolution (use sudo on this script)
```bash
#!/bin/bash

timestamp=$(date +%s)
tcpdump --interface eth0 -nn -w /home/kali/tcpdump-$timestamp.pcap
```

Check storage space/disk usage linux
```bash
df -h
```

bettercap read from file
```
set wifi.handshakes.file /tmp/dup-handshake-dump.pcap
set wifi.source.file /root/bettercap-wifi-handshakes.pcap
wifi.recon on
```

mount s3 bucket to disk
```bash
mkdir /mountpoint
mount-s3 bucket /mountpoint
```

unmount device or s3 bucket
```bash
umount /mountpoint
```

Chrome Exploit <87
https://github.com/r4j0x00/exploits/blob/master/README.md

Print JSONL Schema
```bash
head -n 1 out.json | jq '
  def recurseType: . as $in | if type == "object" then
    reduce keys[] as $key ({}; .[$key] = ($in[$key] | recurseType))
  elif type == "array" then
    map(recurseType)
  else
    type
  end;
  recurseType
'
```

Or for less complicated schema

```bash
jq 'with_entries(.value |= type)' out.json
```

Or to include the length of strings instead

```bash
head -n 1 out.json | jq '
  def recurseTypeOrLength: . as $in | if type == "object" then
    reduce keys[] as $key ({}; .[$key] = ($in[$key] | recurseTypeOrLength))
  elif type == "array" then
    map(recurseTypeOrLength)
  elif type == "string" then
    length
  else
    type
  end;
  recurseTypeOrLength
'
```

Graph network loss
```python
import subprocess
import re
import csv
import matplotlib.pyplot as plt

# Function to run iperf and return the number of lost packets
def run_iperf(kpps):
    cmd = f"sudo /usr/local/bin/iperf -c 10.0.79.99 -u -i 1 -l 16 -b {kpps}kpps -e"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    lost_packets = re.search(r'(\d+)/(?:\d+)\s+\(\d+%\)', result.stdout)
    return int(lost_packets.group(1)) if lost_packets else 0

# Main program
kpps_values = range(1, 401)  # kpps values from 1 to 400
averages = []

# Open a CSV file to log the data
with open('iperf_data.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(['kpps', 'Average Lost Packets'])

    for kpps in kpps_values:
        total_lost = sum(run_iperf(kpps) for _ in range(10)) / 10
        averages.append(total_lost)
        writer.writerow([kpps, total_lost])
        print(f"kpps: {kpps}, Average Lost Packets: {total_lost}")

# Plotting
plt.plot(kpps_values, averages)
plt.xlabel('kpps')
plt.ylabel('Average Lost Packets')
plt.title('Average Lost Packets vs. kpps')
plt.show()
```

