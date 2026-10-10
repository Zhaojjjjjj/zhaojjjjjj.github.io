> 周五下午 4 点，发布窗口 5 点。CI 卡在 checkout 已经 25 分钟，新同学第三次 git clone 失败。运维看了一眼回来只说一句话：".git 10 个 G。"——罪魁祸首是半年前有人把一个 2GB 演示视频 git add 进仓库，之后又改了 10 次。

# Git 大文件：10GB 仓库的解剖与手术

先确诊，两个命令：

```bash
$ du -sh .git
10G

$ git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '/^blob/ {print $3, $4}' | sort -rn | head -5
2147483648 assets/demo.mp4
1879048192 models/weights.bin
```

2GB 的视频改 10 次，1.8GB 的模型权重——三个文件占 .git 的 60%。

# 一、为什么 Git 怕大文件：基因问题，不是 bug

Git 的设计假设是**文本、小文件、高频变更**。每个版本存文件全量快照，再用 delta 压缩去重——文本改一行只多存一点，但**二进制文件的 delta 压缩基本无效**：

```
demo.mp4 改 10 次 → .git 里躺 10 个 2GB = 20GB
```

连锁反应：`git clone` 下载的是**全部历史**（从古至今所有版本）；CI 每次 checkout 都要解包。**仓库里每个操作，都在为那几个大文件交税。**

记住这个结论：Git 是代码版本工具，不是网盘。把二进制大文件塞进 Git，像把家具塞进冰箱——塞得进去，两边都难受。

# 二、Git LFS：止血，不是手术

"从现在起"的大文件，标准答案是 Git LFS。原理优雅：`.git` 里只存几十字节的指针，大文件存 LFS 服务器：

```
$ cat assets/demo.mp4   # .git 里实际存的东西
version https://git-lfs.github.com/spec/v1
oid sha256:4d7a8f3c2e1b...
size 2147483648
```

对开发者透明，GitHub/GitLab 原生支持。但三缺点：**配额**（GitHub 免费版 1GB 存储+1GB/月流量，视频团队几天烧完）；**CI 鉴权**（私有仓库拉 LFS 要 token，`batch response: Authorization error` 是经典坑）；**只管未来**——历史里那 10 个 2GB 还躺着。

关键细节：已有大文件迁进 LFS，`git lfs migrate import --include="*.mp4" --everything`——注意 `--everything` 重写**所有分支**历史，commit hash 全变，团队重 clone。**LFS 迁移和历史清理在"重写历史"上没有本质区别**，区别只在：前者把大文件变指针留历史（可追溯），后者彻底删除（更干净更小）。选哪个，看你在乎"可追溯"还是"体积"。

# 三、历史手术：filter-repo，和不动刀的微创方案

污染了的历史，重写。现代工具是 `git-filter-repo`（`git filter-branch` 的官方继任者）：

```bash
pip install git-filter-repo
# 一刀切：删历史中所有 >100MB 的文件（.git 10GB → 800MB）
git filter-repo --strip-blobs-bigger-than 100M --force
# 精细：只删特定文件
git filter-repo --path assets/demo.mp4 --invert-paths --force
```

**重写历史 = 所有 commit hash 改变**：全员删库重 clone、open 的 PR 重提、挑没人的时间干。操作顺序：先 `git clone --mirror` 备份，等所有人 push 完再动手，强制 push，发公告。漏一步就有人丢代码。

不想动刀？Git 近年的两个"微创"方案：

```bash
# Partial clone：只下 commit 和 tree 结构，blob 按需下载
git clone --filter=blob:none <url>
# Sparse checkout：monorepo 里只检出自己的目录
git sparse-checkout init --cone
git sparse-checkout set apps/web
```

不治本——服务器上 10GB 还在——但**把历史包袱从每个人的日常操作里摘出去**，性价比极高。适合"手术排期中，先让大家跑起来"的过渡期。

# 四、釜底抽薪：别放 Git 里，和让钩子说"不"

有些文件从一开始就不该进 Git：模型权重去 HuggingFace Hub/S3，视频素材去 CDN/OSS，构建产物去 CI Artifacts，大数据集用 DVC（"Git 管代码，DVC 管数据"，`dvc add` 生成指针文件，`dvc push` 推数据到 S3，版本体验和 Git 对齐）。

如果膨胀根源是"什么都往一个仓库塞"，判断标准很简单：**这两部分代码的变更节奏一样吗？** 一天发三版的 UI 和一季度更新的模型权重硬塞一起，互相交税。按疼痛程度渐进：先 sparse checkout 止痛，再评估拆分——**别为架构洁癖而拆，为具体的痛而拆。**

治完病要防复发。靠"自觉"最不可靠——半年后一定有新人 `git add` 2GB 视频。让钩子说"不"：

```bash
# .git/hooks/pre-commit
oversized=$(git diff --cached --name-only --diff-filter=A \
  | xargs -I{} sh -c 'test -f "{}" && [ $(stat -c%s "{}") -gt 52428800 ] && echo "{}"')
if [ -n "$oversized" ]; then
  echo "文件超过 50MB，请走 Git LFS 或外部存储"
  exit 1
fi
```

用 pre-commit 框架版本化进仓库，新同学 clone 自动就有。**机制防住 95% 误操作，共识解决剩下 5% 的"我偏要"**（走例外审批）。

# 结语

那个周五晚上，手术做完：`.git` 10GB→800MB，CI checkout 25 分钟→40 秒。复盘会立了三条规矩沿用至今：**LFS 管现在，外部存储管未来，钩子管人心。**

Git 大文件治理从来不是技术难题——工具都成熟好多年了。真正的难题是**时机**：总在仓库 10GB、CI 卡死、发布窗口被错过的那个周五下午才想起来。`du -sh .git` 只要 5 秒，现在就去看一眼——超过 1GB，这篇就是写给你的。

##{"timestamp":1769659200}