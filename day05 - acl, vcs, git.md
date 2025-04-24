# ACL

```
sudo apt install acl
setfacl --help
getfacl <dir>

// -R: recursive, -m: modify permission, u: user_based_permissions
setfacl -R -m u:ubuntu:rwx devops1
```

# VCS / Git

```bash
git init
git status
git add .
git rm --cached <filename>


git restore --stage index.html
git restore index.html

git revert

```

questions

- how to untrack / remove from history
