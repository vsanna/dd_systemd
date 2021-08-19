## init.d
1. put hello.sh as `/usr/lib/hello/hello.sh`
2. put init.d/myservice as `/etc/init.d/hello`
3. `$ runlevel` to see current runlevel
4. `$ sudo chkconfig --level {current runlevel} hello on`
5. `$ chkconfig`


## servicectl
1. put hello.sh as `/usr/lib/hello/hello.sh`
2. put systemd/hello.service as `/etc/systemd/system/hello.service`
3. `$ systemctl get-default` to see current target
4. `$ cat /etc/systemd/system/hello.service` and see value of `WantedBy` which is same as what we see in step 3
5. `$ systemctl list-units --all`