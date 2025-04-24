# bash shell scripting

`cat /etc/passwd`

0 = root
1-999 = /usr/sbin/nologin (cannot login, but can run process)
1000 = normal users

/etc/profile => user/root
/etc/bash.bashrc => user/root
~/.profile => user
~/.bashrc => user

cat /etc/shells
env

### Profile

`cat .profile`

```
$home/bin
$home/.local/bin
```

"home" is user's home

=> exec in bin folder in home allows the bash to run the script
custom binary

```bash
#### file: example ####

#!/bin/bash
echo  " I am executable"
```

`chmod +x example`

> profile and /etc/profile same for any shell?

bash completion
cat /etc/profile.d/

paths defined
cat /etc/login.defs

## Temp var

export name=bishal
echo $name
unset name

# Permanent var

nano .bashrc
add > export name=bishal
`tail -n 1 .bashrc`

# Bash sript

```bash
#!/bin/bash
name=bishal
echo  "I am : $name"  # variable
echo "who are you?"
read name             # read var
echo "hello: $name"
surname=$1
echo "surname: $surname"  # pass param - example heuju
```

> example heuju
