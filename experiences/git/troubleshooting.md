# Git 问题解决经验

> Git 常见问题解决相关经验
>
> 包含：冲突解决、回滚操作、历史修改、误操作恢复等

---

## 使用 git-filter-repo 重写提交历史（清除敏感信息）

**场景**：需要从 Git 历史中批量替换作者名称/邮箱，清除敏感信息（如真实姓名、公司邮箱）

**日期**：2026-01-29

**问题背景**：
- 提交历史中包含不希望公开的作者信息
- 需要将所有历史提交的作者统一替换为新的身份信息

### 解决方案

#### 1. 安装 git-filter-repo

```powershell
pip install git-filter-repo
```

> ⚠️ 注意：`git-filter-repo` 比 `git filter-branch` 更快、更安全，是官方推荐的历史重写工具。

#### 2. 创建邮箱映射文件（mailmap）

创建 `mailmap.txt`，格式为：`新名称 <新邮箱> 旧名称 <旧邮箱>`

```text
<新用户名> <新邮箱@example.com> <旧用户名> <旧邮箱@company.com>
```

> 📌 **格式要点**：新信息在前，旧信息在后（与 `.mailmap` 文件格式一致）

#### 3. 执行历史重写

```powershell
git filter-repo --mailmap mailmap.txt --force
```

**命令说明**：
- `--mailmap`：指定映射文件
- `--force`：强制执行（跳过"仓库不是全新克隆"的警告）

#### 4. 重新添加远程仓库

> ⚠️ `git-filter-repo` 会自动移除 `origin` 远程，防止误推送

```powershell
git remote add origin https://github.com/<用户名>/<仓库名>.git
```

#### 5. 强制推送到远程

```powershell
git push origin main --force
```

> ⚠️ **警告**：强制推送会覆盖远程历史，协作仓库需提前通知所有成员！

### 验证结果

```powershell
# 查看所有提交的作者信息
git log --format="Author: %an Email: %ae" --all
```

### 常见问题

| 问题 | 解决方案 |
|------|---------|
| 映射没生效 | 检查 mailmap 格式，确保新信息在前、旧信息在后 |
| 编码问题（中文乱码） | 使用 `Out-File -Encoding utf8NoBOM` 保存映射文件 |
| 远程被移除 | 这是正常行为，重新 `git remote add` 即可 |
| 推送失败 | 确认使用 `--force` 参数，并检查远程仓库权限 |

### 完整脚本示例

```powershell
cd "目标仓库目录"

# 1. 创建映射文件（根据实际情况替换占位符）
@"
<新用户名> <新邮箱@example.com> <旧用户名> <旧邮箱@company.com>
"@ | Out-File -Encoding utf8NoBOM mailmap.txt

# 2. 执行重写
git filter-repo --mailmap mailmap.txt --force

# 3. 重新添加远程
git remote add origin https://github.com/<用户名>/<仓库名>.git

# 4. 设置代理（如需翻墙）
$env:HTTPS_PROXY='http://127.0.0.1:7890'

# 5. 强制推送
git push origin main --force

# 6. 清理临时文件
Remove-Item mailmap.txt

# 7. 更新本地 Git 配置（避免后续提交再用旧信息）
git config user.name "<新用户名>"
git config user.email "<新邮箱>"
```

### 相关命令

- 查看提交者统计：`git shortlog -sne --all`
- 只替换邮箱：`git filter-repo --email-callback 'return email.replace(b"<旧邮箱>", b"<新邮箱>")'`
- 只替换名称：`git filter-repo --name-callback 'return name.replace(b"<旧名称>", b"<新名称>")'`

### 注意事项

1. **备份优先**：重写历史前建议先备份仓库
2. **协作通知**：多人协作仓库需通知所有成员重新克隆
3. **CI/CD 影响**：可能需要更新依赖该仓库的自动化流程
4. **不可逆操作**：一旦强制推送，旧历史将无法恢复（除非有备份）

---


---

## Git HTTPS 拉取失败，改用 SSH 协议解决

**日期**：2026-01-30  
**标签**：#git #https #ssh #网络 #connection-reset  
**状态**：✅ 已验证  
**适用版本**：Git 2.x+

**问题/场景**：

在已存在的 Git 仓库执行 `git pull` 或 `git fetch` 时报错：
- `fatal: unable to access 'https://xxx.git/': Recv failure: Connection was aborted`
- `fatal: unable to access 'https://xxx.git/': Recv failure: Connection was reset`

但将仓库**克隆到新位置却可以成功**，说明网络本身没问题。

**排查步骤**：

1. **确认网络连通性**：
   ```powershell
   # 测试能否访问 Git 服务器
   Invoke-WebRequest -Uri "https://你的gitlab地址" -Method Head -TimeoutSec 10
   ```

2. **检查仓库根目录**：
   ```powershell
   # 可能在子目录操作，需找到真正的 .git 所在位置
   git rev-parse --show-toplevel
   ```

3. **尝试增大 HTTP 缓冲区**（无效但可尝试）：
   ```powershell
   git config http.postBuffer 524288000
   ```

4. **运行 GC 清理仓库**（无效但可尝试）：
   ```powershell
   git gc --prune=now
   ```

**解决方案/结论**：

问题根因是 **HTTPS 协议在该仓库的连接被异常中断**，可能与以下因素有关：
- 仓库历史较大，HTTPS 传输超时
- 公司/学校网络对 HTTPS Git 流量有限制
- SSL/TLS 握手问题

**最终解决方案：将远程 URL 从 HTTPS 改为 SSH**：

```powershell
# 查看当前远程配置
git remote -v

# 将 HTTPS URL 改为 SSH URL
git remote set-url origin git@你的gitlab地址:命名空间/仓库名.git

# 再次尝试拉取
git fetch origin
git pull
```

**URL 格式对照**：

| 协议 | 格式 |
|------|------|
| HTTPS | `https://gitlab.com/group/repo.git` |
| SSH | `git@gitlab.com:group/repo.git` |

**前提条件**：

使用 SSH 需要先配置 SSH Key：
```powershell
# 生成 SSH Key（如果没有）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 查看公钥内容，添加到 GitLab/GitHub
Get-Content ~/.ssh/id_ed25519.pub

# 测试 SSH 连接
ssh -T git@你的gitlab地址
```

**处理本地修改冲突**：

切换协议后，如果 `git pull` 报本地修改冲突：

```powershell
# 方案1：暂存本地修改
git stash
git pull
git stash pop

# 方案2：放弃本地修改（.meta 文件通常可以放弃）
git checkout -- .
git pull

# 方案3：先提交再合并
git add -A
git commit -m "保存本地修改"
git pull
```

**验证记录**：

- [2026-01-30] 初次记录，来源：实践总结。在公司内网 GitLab 仓库遇到此问题，HTTPS 持续失败，改 SSH 后立即解决

**备注**：

- 如果 SSH 也不通，检查防火墙是否阻止了 22 端口
- 部分网络可能需要使用 SSH over HTTPS（443端口）：`git@ssh.github.com:用户/仓库.git`
- 子模块（submodule）如果也用了不同协议，需要分别修改


---

## 验证记录：Git HTTPS 拉取失败改用 SSH 协议

**日期**：2026-01-30
**验证状态**：✅ 再次验证成功

**问题现象**：
`
git.exe pull --progress -v --no-rebase -- "origin"
fatal: unable to access 'https://xxx.git/': Recv failure: Connection was aborted
`

**解决过程**：
1. 查看当前远程配置：`git remote -v`
2. 将 HTTPS 改为 SSH：`git remote set-url origin git@服务器地址:命名空间/仓库名.git`
3. 执行拉取：`git pull` → 成功
4. 处理本地修改冲突：`git stash` → `git pull` → `git stash pop`

**结论**：
- 公司内网 GitLab 的 HTTPS 连接不稳定时，SSH 协议是可靠的替代方案
- 切换协议后需要处理本地未提交的修改（stash 或 commit）

**补充说明**：
- 终端输出 `fatal: unknown write failure on standard output` 是输出缓冲区问题，不影响 Git 操作结果
