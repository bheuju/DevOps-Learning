25 port

dpkg -query -s vim

### package install

```bash
#!/bin/bash
packagename=$1
dpkg -l | grep $packagename &> /dev/null  # '&> /dev/null' redirects the o/p of cmd to null
if [ $? == 1 ] ; then                     # '$?' = last command success = 0, fail = 1
    echo "$packagename is not installed"
    apt -y install $packagename
else
    echo "$packagename already exists"
fi

# check $packagename is-enabled; enable if not
status=`systemctl is-enabled $packagename`
if [ $status != "enabled" ] ; then
    systemctl enable $packagename
    echo "`systemctl is-enabled $packagename`"
else
    echo "$packagename already enabled"
fi

# systemctl list-units --type=service | grep postfix
```

`etc/postfix/main.cf`
mailtrap email testing > inboxes > inbox integration > others ? postfix
`echo "content" | mail -s "subject" b.heuju@gmail.com`

crontab -e
`*/1 * * * * bash /home/bishal/scripts/logincount.sh`
