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

