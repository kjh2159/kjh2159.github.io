---
title: git Command Collection (for myself)
date: 2026-04-17 14:30:00 +0900
categories: [git, commands]
tags: [git]
---

# Collection of git Commands by Category



## Fetching a repo

Clone a Git repository.

```shell
git clone https://github.com/[account]/[repo-name].git
````

To perform various remote actions such as git remote and push, an ACCESS TOKEN is required. In this case, clone can also be done using the ACCESS TOKEN.

```shell
git clone https://[ACCESS_TOKEN]@github.com/[account]/[repo-name].git
```

## Connecting to a remote

### Registering an account

Basically, to connect remotely and perform `commit` and `push`, you need to register your account and enter your own token.

```shell
git config --global user.name [account]
```

### Connecting to a remote

`origin` is commonly used as the name of a remote repository. It does not necessarily have to be origin.

```shell
git remote add origin https://github.com/[account]/[repo-name]
```

In this case as well, you can connect using an ACCESS TOKEN.

```shell
git remote add origin https://[ACCESS_TOKEN]@github.com/[account]/[repo-name]
```

### Removing a remote connection

```shell
git remote remove origin
```

## Uploading files

### Adding files

```shell
git add .
git add -A
git add -p #patch
```

### Removing a file from the index

```shell
git restore --staged [filename]
```

### Committing

I find the first commit method more convenient. If you commit using the first method, the `vim editor` opens, where the first line is automatically recognized as the `commit title`, and after one blank line, the rest is automatically recognized as the `commit content`.

```shell
git commit
```

```shell
git commit -m "[commit message]"
```

### Uploading

```shell
git push -u origin [branch name]
```

## Branch

### NEW Branch

`name` is the name of the new branch to create, and the branching point is the name of the branch to branch from.

```shell
git branch [name] [branching point]
```

### DELETE Branch

```shell
git branch -D [name]
```

### Renaming

```shell
git branch -m [old name] [new name]
```

### Switching Branch

```shell
git checkout [name]
```

### Branch status

```shell
git branch -v
```

```shell
git branch --merged
```

```shell
git branch --no-merged
```