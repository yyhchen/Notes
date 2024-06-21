# 在其他电脑中添加 另一个 github 账户
---

### 首先，给本地配置想要添加的 github 账户 生成新的 ssh key

`ssh-keygen -t rsa -C “xxx@xx.com”`

需要注意的是：
- 要重新命名，否则会覆盖之前的 `id_rsa`

结果类似如下：
- 命名为 rsa_private

![text](image.png)


配置本地 `.ssh/` 文件夹下的 `config` 文件：
```shell
# one
Host github_yuehuasolo.com
Hostname ssh.github.com
Port 443
IdentityFile ~/.ssh/id_rsa
ServerAliveInterval 60


# other account
Host github_yyhchen.com
Hostname ssh.github.com
Port 22
IdentityFile ~/.ssh/rsa_private
```

其中， `Hostname` 是统一的，链接用；`Host` 是 `git@xx` 中的后缀，比如后面需要 `clone` 项目的时候，`git clone git@github_yyhchen: 用户名/仓库名.git` 即可


配置完文件后， 用 `git -T git@github_yyhchen.com` 测试，结果如下：
```
Hi yyhchen! You've successfully authenticated, but GitHub does not provide shell access.
```

同理，测试原来的账户也是用 `git -T git@自己配的Host`


<br>


### 其次，需要注意的是，因为有多个账户，所以需要取消原来的全局用户名和邮箱

```shell
git config --global --unset user.name

git config --global --unset user.email

```

之后需要 `clone` 项目，先设置 该文件下 的用户名和邮箱：
```shell
git config --local user.name

git config --local user.email
```

然后再 使用 `git clone git@github_yyhchen.com: 用户名/仓库名.git` 即可 


<br>


### 最后，千万要记住！！！ 如果到原来的电脑上更新仓库，必须先 `git pull origin main` !!!


<br>
<br>
<br>


# 在当前情况新增github仓库

---

1. github 先创好仓库
2. git remote add origin 的时候，这里用上 之前定义的 hostname, 即完整的应该是 `git clone git@github_yyhchen.com: 用户名/仓库名.git` ; 否则会报错！！

<br>
<br>
<br>

# 在当前环境创建仓库出现的一些错误 

在创建新仓库后,新仓库添加额了 `README.md`，本地初始化和提交后使用 `git push -u origin ` 出现错误如下：

```vbnet
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github_yyhchen.com:yyhchen/Model_Deploy_tutorial.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

**这个错误提示是由于远程仓库包含本地没有的提交记录。在这种情况下，你需要先将远程仓库的更改拉取到本地，然后再进行推送。**

<br>

==**但是**== ，直接使用 `git pull origin main` 也会**出现错误**, 而且并没有提交到远程仓库

**错误如下：**
```vbnet
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint: 
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint: 
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
fatal: Need to specify how to reconcile divergent branches.
```

**这个错误提示你有分歧的分支，并且需要指定如何合并它们。这是因为远程仓库的历史和本地仓库的历史已经不同步了**

**解决办法:**
使用 `git pull --rebase origin main` 

**原理是：**
`rebase` 会将当前分支的提交一个一个地提取出来，并在目标分支的最新提交上重新应用。这会生成新的提交对象，历史记录会被重写成一条线性路径，使得提交历史更加简洁

<br>

## 根据上面的操作，但是后面在github上发现了本地用户的提交名

**原因是**，一开始并没有设置用户名和邮箱～ 因为本地两个账户的原因取消了 全局的 `user.name` 和 `user.email`


## 所以！ 在当前的环境创建新的 git 与 github 的仓库时，需要先 `git config --local user.name "yyhchen"` 和 `git config --local user.email xxx"`