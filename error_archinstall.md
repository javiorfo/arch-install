# Before using command archinstall
- edit /etc/systemd/timesyncd.conf
```bash
[Time]
NTP=time.google.com
FallbackNTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
```
-Restart time 
```bash
systemctl restart systemd-timesyncd
```
