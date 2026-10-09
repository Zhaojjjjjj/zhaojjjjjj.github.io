> Git 仓库 10GB，`git clone` 要半小时，CI 每次 checkout 5 分钟。罪魁祸首：有人把 2GB 的视频、模型文件直接 `git add` 了。这篇讲讲 Git 大文件的治理。

## 症状

```bash
$ du -sh .git
10G
$ git rev-list --objects --all | grep -E '\.(mp4|bin|zip)$' | head
a1b2c3d  2.1G  assets/demo.mp4
e4f5g6h  1.8G  models/weights.bin
```

## 为什么 Git 怕大文件

Git 的设计假设：**文本、小文件、高频变更**。每个版本存全量（delta 压缩对二进制无效）：

```
demo.mp4 改了 10 次 → .git 里存 10 个 2GB = 20GB
```

`git clone` 要下载全部历史，`git log` 要遍历，**每个操作都慢**。

## 方案一：Git LFS（标准答案）

```bash
# 安装后，大文件走 LFS
git lfs install
git lfs track "*.mp4" "*.bin" "*.zip"
git add .gitattributes

# 原理：.git 里只存指针（几十字节），大文件存 LFS 服务器
$ cat assets/demo.mp4
version https://git-lfs.github.com/spec/v1
oid sha256:4d7a...
size 2147483648
```

**优点**：透明，`git clone` 自动拉取，GitHub/GitLab 原生支持。
**缺点**：LFS 有配额（GitHub 免费 1GB），超了要花钱；CI 要装 `git-lfs`。

## 方案二：历史手术（已经污染了怎么办）

```bash
# 用 git-filter-repo（git filter-branch 的继任者）重写历史
pip install git-filter-repo
git filter-repo --strip-blobs-bigger-than 100M --force

# 效果：所有 >100M 的文件从历史中彻底删除
# .git 从 10GB → 800MB
```

**警告**：重写历史 = 所有 commit hash 变了，**团队所有人要重新 clone**。提前通知，选个没人的时间干。

```bash
# 更精细：只删特定文件
git filter-repo --path assets/demo.mp4 --invert-paths --force
```

## 方案三：别放 Git 里（釜底抽薪）

| 文件类型 | 放哪 |
|---|---|
| 模型权重 | HuggingFace Hub / S3 |
| 视频/图片素材 | CDN / OSS |
| 构建产物 | Artifacts（CI 产物库） |
| 大数据集 | DVC（Data Version Control） |

**DVC** 值得一提：Git 管代码，DVC 管数据，用法和 Git 一样：

```bash
dvc add data/train.csv  # 生成 .dvc 指针文件，git 只管这个
dvc push  # 数据推到 S3
```

## 防患：pre-commit 钩子

```bash
# .git/hooks/pre-commit
#!/bin/bash
# 拒绝 >50MB 的文件进仓库
large=$(git diff --cached --name-only | xargs ls -la 2>/dev/null | awk '$5 > 52428800')
if [ -n "$large" ]; then
  echo "❌ 文件太大，用 Git LFS 或放外部存储："
  echo "$large"
  exit 1
fi
```

## 一句话总结

Git 大文件治理三步：**LFS 管现在、filter-repo 治历史、外部存储管未来**。记住：Git 是代码版本工具，不是网盘。下次有人 `git add` 一个 2GB 的视频，pre-commit 会替你说"不"。

##{"timestamp":1769659200}