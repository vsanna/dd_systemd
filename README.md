## init.d
1. put hello.sh as `/usr/lib/hello/hello.sh` and `$ chmod a+x hello`
2. put init.d/hello as `/etc/init.d/hello`
3. `$ runlevel` to see current runlevel
4. `$ chkconfig --add {servicename}`
5. `$ sudo chkconfig --level {current runlevel} hello on`
	- rc script has chkconfig magic comment and it defines runlevel, so actually this step is not necessary
6. `$ chkconfig`
7. `$ service hello status`


## servicectl
1. put hello.sh as `/usr/lib/hello/hello.sh` and `$ chmod a+x hello`
2. put systemd/hello.service as `/etc/systemd/system/hello.service`
3. `$ systemctl get-default` to see current target
4. `$ cat /etc/systemd/system/hello.service` and see value of `WantedBy` which is same as what we see in step 3
5. `$ systemctl list-unit-files --type=service | grep hello` to see the service is recognized by systemctl or not
6. `$ systemctl enable hello` to enable it (= create symlink)
7. `$ systemctl status hello`
