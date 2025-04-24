# Agile

- epic
- story

[x] Hello

Agile Manifesto

Linux Bootup process

---

cat /var/log/syslog
cat /var/log/syslog | grep -A 3 -B 3 Down

cat /etc/os-release

```bash
echo hello worl > example
cat example
sed -i 's/worl/world/' example
cat example

cat example | awk '{ print $2 }'
```

```
`>>` append
`>` write
```

# Install service

```
sudo apt install nginx -y
sudo apt list --installed | grep nginx # check installed
sudo dpkg -l | grep nginx # check installed
nginx -v

sudo systemctl is-enabled nginx
sudo systemctl is-active nginx
sudo systemctl status nginx
sudo systemctl status nginx | grep loaded
cat /usr/lib/systemd/system/nginx.service
```

```ini
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

services:

```bash
[Unit]
[Service]
[Install]
```

```
systemctl set-default multi-user.target
systemctl get-default
```

---

### ??? graphical / multi-user target --- study =======

`systemctl list-units | grep ssh`

```
systemctl status ssh
systemctl status sshd # doesn't work without "enabled"
" alias actives only when service is enabled by **systemctl enable ssh** "
```

```
cat /usr/lib/systemd/system/ssh.service
```

```ini
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
**Alias=sshd.service`**
```

---

# Nginx Configuration

```bash
cd /etc/nginx
cat nginx.conf
cat nginx.conf | grep -v ^# | more
```

```
user

```
