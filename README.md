# 如何向 Android 通用内核提交补丁

1. 最佳做法：将所有更改都提交到上游 Linux。如果合适，请反向移植到稳定版本。

这些补丁将自动合并到相应的通用内核中。如果补丁已在上游 Linux 中，请发布符合以下补丁要求的补丁反向移植。

- 不要向上游发送仅包含符号导出的补丁。要考虑用于上游 Linux，

`EXPORT_SYMBOL_GPL()` 的添加需要使用符号的树内模块化驱动程序 - 因此请将

新驱动程序或对现有驱动程序的更改包含在与导出相同的补丁集中。

- 在向上游发送补丁时，提交消息必须包含一个清晰的案例，说明为什么需要该补丁以及该补丁对社区有益。启用树外驱动程序或功能并不是一个有说服力的案例。

2. 不太好：从树外开发补丁（从上游 Linux 的角度来看）。除非这些补丁是修复 Android 特有的错误，否则它们不太可能被接受，除非它们已与 kernel-team@android.com 协调。如果您想继续，请发布符合以下补丁要求的补丁。

# 通用内核补丁要求

- 所有补丁都必须符合 Linux 内核编码标准并通过 `scripts/checkpatch.pl`
- 补丁不得破坏 arm、arm64、x86、x86_64 架构的 gki_defconfig 或 allmodconfig 构建
（请参阅 https://source.android.com/setup/build/building-kernels）
- 如果补丁不是从上游分支合并的，则必须使用补丁类型标记主题：
`UPSTREAM:`、`BACKPORT:`、`FROMGIT:`、`FROMLIST:` 或 `ANDROID:`。
- 所有补丁都必须有一个 `Change-Id:` 标签（请参阅 https://gerrit-review.googlesource.com/Documentation/user-changeid.html）
- 如果已分配 Android 错误，则必须有一个 `Bug:` 标签。
- 所有补丁都必须有一个由作者和提交者签署的 `Signed-off-by:` 标签

根据补丁类型，下面列出了其他要求

## 从主线 Linux 反向移植的要求：`UPSTREAM:`、`BACKPORT:`

- 如果补丁是从 Linux 主线中挑选出来的，没有任何变化
- 用 `UPSTREAM:` 标记补丁主题。
- 使用 `(cherry picking from commit ...)` 行添加上游提交信息
- 示例：
- 如果上游提交消息是
```
来自上游的重要补丁

这是重要补丁的详细描述

签名人：Fred Jones <fred.jones@foo.org>
```
>- 那么 Joe Smith 会将通用内核的补丁上传为
```
上游：来自上游的重要补丁

这是重要补丁的详细描述

签名人：Fred Jones <fred.jones@foo.org>

Bug：135791357
Change-Id：I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
(cherry picking from commit c31e73121f4c1ec41143423ac6ce3ce6dafdcec1)
签名人：Joe Smith <joe.smith@foo.org>
```

- 如果补丁需要对上游版本进行任何更改，请使用 `BACKPORT:`
而不是 `UPSTREAM:` 标记补丁。
- 使用与 `UPSTREAM:` 相同的标签
- 在 `(cherry picking from commit ...)` 行下添加有关更改的注释
- 示例：
```
BACKPORT：来自上游的重要补丁

这是重要补丁的详细描述

签名人：Fred Jones <fred.jones@foo.org>

Bug：135791357
Change-Id：I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
(cherry picking from commit c31e73121f4c1ec41143423ac6ce3ce6dafdcec1)
[joe：解决了 drivers/foo/bar.c 中的小冲突 ]
签名人：Joe Smith <joe.smith@foo.org>
```

## 其他反向移植的要求：`FROMGIT:`， `FROMLIST:`，

- 如果补丁已合并到上游维护者树中，但尚未
合并到 Linux 主线
- 使用 `FROMGIT:` 标记补丁主题
- 添加补丁来源信息为 `(cherry picking from commit <sha1> <repo> <branch>)`。这
必须是一个稳定的维护者分支（未重新定基，因此不要使用 `linux-next` 作为示例）。
- 如果需要更改，请使用 `BACKPORT: FROMGIT:`
- 示例：
- 如果维护者树中的提交消息是
```
来自上游的重要补丁

这是重要补丁的详细描述

签名人：Fred Jones <fred.jones@foo.org>
```
>- 那么 Joe Smith 会将通用内核的补丁上传为
```
FROMGIT：来自上游的重要补丁

这是重要补丁的详细描述

签名人：Fred Jones <fred.jones@foo.org>

Bug：135791357
（从提交 878a2fd9de10b03d11d2f622250285c7e63deace 中挑选
https://git.kernel.org/pub/scm/linux/kernel/git/foo/bar.git test-branch)
Change-Id： I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
签名人：Joe Smith <joe.smith@foo.org>
```

- 如果补丁已提交给 LKML，但未被接受

