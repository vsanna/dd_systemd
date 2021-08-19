## note
- daemon:
	- UNIX系にある常駐プログラム. 常にプロセスが立ち上がっていてリクエストを待っている
		- ref: JavaのDaemon thread
- init.d:
	- runlevelに応じて起動/停止
		- runlevel:
			- 動作モードのこと. 各レベルごとにどのサービスが起動する/しないを決める
			- `$ runlevel` or `$ systemctl get-default`
			- 0-6の7段階
				- 0: poweroff: 何も動かない
				- 1: rescue: rootでもログインできないときに使う
				- 6: システム再起動
			- `$ chkconfig` で登録されているサービスのランレベル別on/offを一覧
		- rcファイル: `/etc/rc3.d/S10network -> ../init.d/network`
			- start|stop|restart|statusなどのoptionに応じて何をするかを記述するscriptのこと.デーモンを管理するscript
			- rc*.d配下にsymlinkを置くことで、どのランレベルでどのサービスを起動するかを指定
			- (S|K)(priority(int))(scriptname)
				- S: start, K: kill(=stop)
					- **init.d/配下のscriptにわたす引数が start or stop にになるのをここで指定**
				- priority: 小さいほど優先度が高い = 先に実行される
				- scriptname -> 実行するscriptの名前
			- 各サービス、別runlevelでは別優先度を割り当てられる
	- デーモンの作り方
		1. 何はなくともまずはデーモンで実行する対象となる`実行ファイル`を用意. ex: apache
		2. /etc/init.d/ 配下に start/stop/restart というコマンド引数を受け付けるデーモン起動/停止スクリプトを書く
			- つまり、 /etc/init.d/hoge start|stop と打っても起動時と同じことが行える
		3. /etc/rc*.d/配下に そのinit.d配下の起動/停止scriptへのsymlinkを貼る。
			- `update-rc.d` を使ってsymlinkはる!
				-> 実際には `$ sudo chkconfig --level 6 network off|on`
			- 命名規則が (S|K)(priority(int))(scriptname)
	- 余談
		- macOSではlaunchdがinit.dに当たる
		- 手順
			1. /Library/LaunchDaemons に init.d/配下のscriptに相当する 通称launchd.plist を作る
				- ex. /Library/LaunchDaemons/com.google.keystone.daemon.plist
			2. `$ launchctl load /Library/LaunchDaemons/yours.plist` で読み込み. 次回以降OS起動時に参照される
- service
	- `$ service xxxx start|stop|status`
	- 実際には /etc/init.d 配下をうまく管理するためのwrapper ツール
		- `/sbin/service` で中身を見れる. そこまで大きくない
	- vs init.d
		- /etc/init.d/xxxx start -> 環境変数がすべて引き継がれる
		- service xxxx start -> PATHとTERMのみ引き継がれる
	- よく使うoption
		- `$ service --status-all`
- systemctl
	- systemctl = systemdを管理するコマンド
		- systemd = CentOS7からinitに変わってPID=1として動くプロセス
			- `ps aux | awk '($2==1){ print $0 }'`
			- `root         1  0.0  0.5 125688  5616 ?        Ss   02:14   0:02 /usr/lib/systemd/systemd --switched-root --system --deserialize 21`
		- init = init.dの仕組みそのもの. ランレベルに応じたデーモンの起動を担う仕組みでPID1として動いていた
			- but, rcスクリプトの複雑さ, 並列処理の非対応, 親子プロセスの制御が苦手, などの課題もあった
	- target: ほぼrunlevelに相当する概念.
		- `$ ls -la /lib/systemd/system/runlevel*.target` で対応関係を確認できる
		- `$ systemctl get-default` で確認
	- unit: ほぼrc scriptに相当する概念
		- ex: `/usr/lib/systemd/system/crond.service`
			- 注意! `/usr/lib/systemd/system/` のファイルは編集しない
			- `/etc/systemd/system` にファイルを追いて上書きする
	- how to use:
		- `$ systemctl start|stop|restart|status xxxxx` ... 順番が微妙に変わってる!
		- serviceとの違いは?
			- service -> /etc/init.d/配下を読んでいたが、systemctlは **バイナリコマンド** を呼び出す
	- chkconfingの代わり
		- `$ systemctl list-unit-files` or `$ systemctl list-units --all`
		- `$ systemctl enable|disable {service}`

```
[ec2-user@ip-172-31-33-42 rc.d]$ ls -la rc1.d
合計 0
drwxr-xr-x  2 root root  45  7月 26 23:53 .
drwxr-xr-x 10 root root 127  7月 26 23:52 ..
lrwxrwxrwx  1 root root  20  7月 26 23:53 K50netconsole -> ../init.d/netconsole
lrwxrwxrwx  1 root root  17  7月 26 23:53 K90network -> ../init.d/network
[ec2-user@ip-172-31-33-42 rc.d]$ ls -la rc2.d
合計 0
drwxr-xr-x  2 root root  45  7月 26 23:53 .
drwxr-xr-x 10 root root 127  7月 26 23:52 ..
lrwxrwxrwx  1 root root  20  7月 26 23:53 K50netconsole -> ../init.d/netconsole
lrwxrwxrwx  1 root root  17  7月 26 23:53 S10network -> ../init.d/network
```

```
[ec2-user@ip-172-31-33-42 rc.d]$ ls -la /lib/systemd/system/runlevel*.target
lrwxrwxrwx 1 root root 15  7月 26 23:52 /lib/systemd/system/runlevel0.target -> poweroff.target
lrwxrwxrwx 1 root root 13  7月 26 23:52 /lib/systemd/system/runlevel1.target -> rescue.target
lrwxrwxrwx 1 root root 17  7月 26 23:52 /lib/systemd/system/runlevel2.target -> multi-user.target
lrwxrwxrwx 1 root root 17  7月 26 23:52 /lib/systemd/system/runlevel3.target -> multi-user.target
lrwxrwxrwx 1 root root 17  7月 26 23:52 /lib/systemd/system/runlevel4.target -> multi-user.target
lrwxrwxrwx 1 root root 16  7月 26 23:52 /lib/systemd/system/runlevel5.target -> graphical.target
lrwxrwxrwx 1 root root 13  7月 26 23:52 /lib/systemd/system/runlevel6.target -> reboot.target
```

```
[ec2-user@ip-172-31-33-42 rc.d]$ cat /usr/lib/systemd/system/crond.service
[Unit]
Description=Command Scheduler
After=auditd.service systemd-user-sessions.service time-sync.target

[Service]
EnvironmentFile=/etc/sysconfig/crond
ExecStart=/usr/sbin/crond -n $CRONDARGS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=30s

[Install]
WantedBy=multi-user.target
```

## ref
- http://nw.tsuda.ac.jp/~nitta/lec/daemon/
- https://kanonji.hatenadiary.com/entry/20100621/1277075926
- https://qiita.com/manjiroukeigo/items/2b217ffbb50b119d5f58
- https://milestone-of-se.nesuke.com/sv-basic/linux-basic/systemctl/
- https://www.karakaram.com/how-to-use-systemctl-journalctl/
- https://qiita.com/ledmonster/items/5f2e1633d4124cb978fe

