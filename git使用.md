# git使用

## 问题1：跟踪文件夹

- 如果你想将 `C++_Notes` 文件夹纳入版本控制：

  1. **添加文件到暂存区：**

     ```
     git add C++_Notes/
     ```

  2. **然后提交：**

     ```
     git commit -m "添加C++学习笔记"
     ```

- 在 Git 跟踪的仓库中创建文件夹并跟踪：

​	**Git 不跟踪空文件夹**，所以必须确保：

​		1、文件夹内至少有一个文件

​		2、常见做法是创建 `README.md` 或空的 `.gitkeep` 文件

```
# 1. 创建文件夹
mkdir C++_Notes

# 2. 进入文件夹并创建至少一个文件（Git不跟踪空文件夹）
cd C++_Notes
echo "# C++学习笔记" > README.md
# 或者创建一个空文件：touch .gitkeep

# 3. 返回仓库根目录
cd ..

# 4. 添加文件夹到Git
git add C++_Notes/

# 5. 提交
git commit -m "添加C++学习笔记文件夹"
```

## 问题2：跟踪新复制到文件夹中的文件

1. **复制文件**到 `C++_Notes` 文件夹
2. **检查状态**：`git status` 查看哪些文件是新的
3. **添加文件**：`git add C++_Notes/` 或 `git add .`
4. **提交更改**：`git commit -m "描述"`
5. **推送到远程**（如果需要）：`git push`

**方法1：添加整个文件夹的所有新文件**

```
# 添加 C++_Notes 文件夹中的所有新文件
git add C++_Notes/

# 或者添加所有变更（包括新文件、修改的文件）
git add .
```

**方法2：添加特定文件类型**

```
# 只添加所有的 markdown 文件
git add C++_Notes/*.md

# 或者添加整个仓库中的所有 markdown 文件
git add *.md
```

## 问题3：推送代码到 GitHub

首先，确保你已经在本地完成了代码的提交（即已执行 `git add` 和 `git commit`）。接下来，请按照以下步骤操作：

1. **关联远程仓库**
   你需要告诉 Git 远程仓库在哪里。首先在 GitHub 上创建一个新的空仓库（不要初始化 README 文件），然后使用以下命令进行关联：

   ```
   git remote add origin https://github.com_你的用户名_/你的仓库名.git
   ```

   这里的 `origin` 是远程仓库的别名，之后可以用它来指代这个地址。

2. **推送到远程仓库**
   关联之后，就可以将本地代码推送上去了。如果你本地的主分支名叫 `master`，则使用：

   ```
   git push -u origin master
   ```

   如果分支名叫 `main`，则使用：

   ```
   git push -u origin main
   ```

   参数 `-u` 表示建立上游追踪，下次推送时直接使用 `git push` 即可。

## 问题4：新建分支以后，主分支的文件夹会出现在新分支里面？

这是 **Git 的正常行为**：

- 新分支继承父分支的所有已提交内容
- 未跟踪的文件会出现在所有分支的工作区中
- 只有提交后的文件才真正"属于"特定分支

1. **分支创建机制**

当你创建新分支 `tiger` 时：

```
git branch tiger
git switch tiger
```

**新分支 `tiger` 是基于当前分支（master）创建的**，所以它会包含 master 分支的所有文件。

2. **工作目录共享**

- Git 的所有分支**共享同一个工作目录**
- 当你切换分支时，Git 会自动更新工作目录以匹配该分支的内容
- 但新创建的文件（如 `cat1.html`, `cat2.html`）在没有提交前，**属于未跟踪状态**，会出现在所有分支中
- 文件只有在特定分支提交后，才属于该分支

## 问题5：把新分支推送到Github上面

- 确保**本地当前位于 `tiger` 分支**上。
- **确保代码已提交**：在执行 `git push` 之前，请确认你在 `tiger` 分支上的所有修改都已经使用 `git commit` 提交了。`git push` 只推送提交（commit），不会推送未保存的本地修改。
- 推送本地分支到远程仓库的核心命令是 `git push`。对于首次推送 `tiger` 分支，最推荐的做法是使用 `-u`（或 `--set-upstream`）选项，这可以建立本地分支与远程分支的追踪关系。

```
git push -u origin tiger
```

## 问题6：在github上tiger分支创建文件，如何拉取到本地对应的分支

- 确保本地已经切换到了 `tiger` 分支**。

- 在本地的 `tiger` 分支后，就可以从远程仓库拉取更新了。最常用的命令是：

  ```
  git pull origin tiger
  ```

## 问题7：合并分支

1. 将其他分支合并到当前分支

```
# 假设当前在 master 分支，想合并 tiger 分支
git merge tiger
```

2. 使用提交信息（避免进入编辑器）

```
git merge tiger -m "合并tiger分支的功能"
```

PS：合并后的清理————删除已合并的分支

```
# 删除本地分支
git branch -d tiger

# 删除远程分支
git push origin --delete tiger
```

## 问题8：修改文件

- ##### 新文件 vs 已跟踪文件的修改

  - **新文件**：需要先 `git add` 然后 `git commit`
  - **已跟踪文件的修改**：直接 `git add` 然后 `git commit`，或使用 `git commit -am`

  记住这个简单的工作流程：**修改 → 添加 → 提交 → 推送**

## 问题9：合并相关的问题

- 合并分支

  - 将其他分支合并到当前分支

  ```
  # 假设当前在 master 分支，想合并 tiger 分支
  git merge tiger
  ```

  - 使用提交信息（避免进入编辑器）

  ```
  git merge tiger -m "合并tiger分支的功能"
  ```

  - PS：合并后的清理————删除已合并的分支

  ```
  # 删除本地分支
  git branch -d tiger
  
  # 删除远程分支
  git push origin --delete tiger
  ```

- 如果两个分支中有同名文件合并时会发生冲突？
  1. 什么时候会产生冲突？
     - 如果两个分支对**同一文件的相同位置**都有修改，才会产生冲突
     - Git 只会在以下情况报告冲突：
       - 两个分支修改了**同一文件的相同行**
       - 一个分支修改了文件，另一个分支删除了该文件
       - 重命名/移动文件产生歧义
  2. Git 的智能合并策略：
     - 一个文件是空的
     - 另一个文件有内容
     - 这属于"不重叠的修改"，可以安全自动合并
     - 当两个分支对同一个文件的修改不重叠时（比如一个分支在文件末尾添加行，另一个分支在文件开头添加行），Git可以自动合并。

## 问题10：删除文件，并git移除跟踪

1. **删除单个文件并提交**

```
git rm filename.html
git commit -m "删除 filename.html"
```

2. **删除所有 .html 文件**

```
git rm *.html
git commit -m "删除所有 HTML 文件"
```

3. **如果文件已在 .gitignore 中，但仍想从 Git 中删除**

```
git rm --cached *.html
git commit -m "从 Git 中移除 HTML 文件（保留本地文件）"
```

4. **递归删除所有目录中的 .html 文件**

```
git rm **/*.html
git commit -m "递归删除所有 HTML 文件"
```

**5.如果只想删除远程文件但保留本地文件**

```
# 从 Git 中删除但保留本地文件
git rm --cached *.html
git commit -m "从版本控制中移除 HTML 文件（保留本地）"
git push
```

**查看将要删除的文件（预览）**

```
git rm -n *.html  # -n 参数用于预览，不会实际删除
```

**强制删除（即使文件有修改）**

```
git rm -f *.html
```

**删除目录**

```
git rm -r directory_name/
```

**如果需要同步删除远程仓库里的文件**

```
# 推送到默认远程分支（通常是 main 或 master）
git push origin main

# 或者如果你在当前分支
git push
```

