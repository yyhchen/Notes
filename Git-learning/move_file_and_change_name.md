# 移动 git 管理后的文件，以及修改 github仓库名对仓库影响

---


### 移动文件

```bash
git mv file file_new
```

这种方式不会出现一些bug，不会有远程和本地的冲突；后续就还是正常的 git add + commit 操作


<br>
<br>



### 修改仓库名

在 GitHub 上更改了仓库的名称后，本地仓库的远程 URL 仍然指向旧的仓库名称。为了同步本地仓库与 GitHub 上的新仓库名称，需要更新本地仓库的远程 URL。以下是具体步骤：

1. **检查当前的远程 URL**：
   首先，检查本地仓库当前的远程 URL，确保它指向的是旧的仓库名称。
   ```sh
   git remote -v
   ```

2. **更新远程 URL**：
   更新远程 URL 以指向新的仓库名称。假设新仓库 URL 是 `https://github.com/username/new-repo-name.git`，使用以下命令：
   ```sh
   git remote set-url origin https://github.com/username/new-repo-name.git
   ```

3. **验证更新**：
   再次检查远程 URL，确保它已经更新为新的仓库名称。
   ```sh
   git remote -v
   ```

4. **拉取最新更改**：
   如果在 GitHub 上进行了一些更改，并且希望将这些更改拉取到本地仓库，可以使用以下命令：
   ```sh
   git pull origin main
   ```


* 注意：本地仓库名并没有被改变，只是改变了 github 远程仓库的链接，上面的操作是为了一些代码合并等问题不会有冲突和报错