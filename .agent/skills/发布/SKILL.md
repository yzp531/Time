---
name: 发布
description: 发布流程自动化：更新系统索引、推送当前分支、创建新版本并推送到远端。无变更时直接返回。
short_description: 一键完成索引更新、分支推送、版本递增与远端推送。
short_description_zh: 一键完成索引更新、分支推送、版本递增与远端推送。
version: 3
updated: 2026-09-23T12:20:00Z
---

# 发布

一键完成发布流程：更新文档索引 → 推送当前分支 → 计算下一版本 → 创建新分支 → 推送到远端。

## 版本规则

- 格式 `vX.Y.Z`，X 范围 0–9，Y 范围 0–99，Z 范围 1–99
- 递增顺序：Z 优先，Z 达 99 后进位到 Y（Y 同时归零、Z 归 1）；Y 达 99 后进位到 X（X 归零、Y 归零、Z 归 1）
- 分支名与版本号一一对应（如 `v0.0.23`）

## 前置检查（无变更则直接返回）

执行任何操作前，先检查：

```bash
# 1. 是否有未提交改动
git status --porcelain
# 2. 当前分支是否已推送到远端且无落后
git rev-list --count origin/$(git branch --show-current)..$(git branch --show-current)
```

**如果 `git status --porcelain` 为空 且 `git rev-list` 计数为 0**，说明无新变更、分支已是最新，直接返回，不执行后续流程。

## 前置条件

- 工作目录为项目根目录（`stocks.i-xx.top`）
- 当前分支名即为当前版本号（如 `v0.0.22`）
- 所有变更已提交（`git status` 无未提交改动）

## 流程

### 1. 更新系统索引

```bash
python3 agent/check_docs.py
```

- 校验 `agent/FUNCTIONS.index.md`、`agent/fix.md` 等文档的行数预算与引用存在性
- 退出码非 0 表示存在问题，需先修复再继续

### 2. 推送当前分支

```bash
git push origin refs/heads/$(git branch --show-current):refs/heads/$(git branch --show-current)
```

### 3. 计算下一版本

从当前分支名解析 `vX.Y.Z`，按规则递增：

```python
def next_version(current: str) -> str:
    """current like 'v0.0.22' → 'v0.0.23'"""
    v = current[1:].split('.')
    x, y, z = int(v[0]), int(v[1]), int(v[2])
    z += 1
    if z > 99:
        z = 1
        y += 1
    if y > 99:
        y = 0
        x += 1
    if x > 9:
        raise ValueError('版本号已达上限 v9.99.99')
    return f'v{x}.{y}.{z}'
```

### 4. 创建新版本分支并推送

```bash
NEW_VER=$(python3 -c "
cur = '$(git branch --show-current)'
x, y, z = (int(p) for p in cur[1:].split('.'))
z += 1
if z > 99: z, y = 1, y + 1
if y > 99: x, y = x + 1, 0
if x > 9: raise SystemExit('版本号已达上限 v9.99.99')
print(f'v{x}.{y}.{z}')
")
git checkout -b "$NEW_VER"
# 分支名与 tag 同名，必须写完整 refspec，否则报 src refspec matches more than one
git push origin refs/heads/"$NEW_VER":refs/heads/"$NEW_VER"
```

分支与 tag 都已存在于本仓库时，tag 用同规则推送：

```bash
git tag -a "$NEW_VER" -m "<本次发布说明>"
git push origin refs/tags/"$NEW_VER":refs/tags/"$NEW_VER"
```

完成后核对远端：

```bash
git ls-remote --heads origin | grep "$NEW_VER"
git ls-remote --tags origin | grep "$NEW_VER"
```

## 注意事项

- 仅在当前分支已提交干净、索引校验通过后执行
- 推送远端前确认无并发冲突（`git fetch origin` 后再操作）
- 新分支创建后，后续开发在新分支上进行
- **所有 push 都写完整 refspec**（`refs/heads/X:refs/heads/X`、`refs/tags/X:refs/tags/X`）：版本分支与同名 tag 共存，简写 `git push origin vX.Y.Z` 会报 `src refspec matches more than one`
- 先推当前分支、再切新版本分支，旧分支就不会「领先远端」；若已领先，本次提交已存在于新版本分支，可用 `git branch -f <旧分支> origin/<旧分支>` 拉回快照口径
- 第 3、4 步的版本递增逻辑必须一致（`vX.Y.Z` 各自独立解析，**不要**对 `split('.')` 的结果再切一刀）；改一处时同步改另一处
- `check_docs.py` 目前恒报 `AGENTS.md` 行数超限（既有问题，与当次改动无关）：据实说明后继续发布，不要为了过检查擅自删改协作规则
- 脚本报错先读错误信息：`int('')` / `invalid literal` 基本就是版本解析切片写错，按上面两条修正后重算，不要改用猜的版本号