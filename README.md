# git-bisect-find-bug-commit

使用 `git bisect` 命令来找出导致 bug 的 commit 提交记录。

Use git bisect command to find the commit that caused the bug.

## git bisect 是什么

git bisect 二分查找

`git bisect` 命令可以让你通过二分法从一堆 commit 中找出导致 bug 的 commit

## 基础用法

终端运行 `git bisect start` 进入 bisect 模式，通过 `git bisect good` 和 `git bisect bad` 命令给当前的 commit 打上标记，good 表示没有出现 bug，bad 表示出现了 bug。第一个 good 和 bad 标记会成为二分法的起始点。

当第一个 good 和 bad 标记添加好后，会开始 git bisect，会跳转到需要判断的 commit，当你判断完当前的 commit 是否存在 bug 之后，标记上 good or bad，会跳转到下一个需要判断的 commit，直到找到导致 bug 的 commit

执行 `git bisect reset` 退出 bisect 模式

## 全自动用法

手动判断还是太麻烦了，最正确的方式是，写一个针对当前 bug 的测试用例，在 bisect 模式通过 `git bisect run [command]` 运行测试用例，自动判断当前 commit 是否有问题，从而找出 bug commit

### 试一试

`index.js` 文件中，定义了一个 sum 方法，返回参数 a + b 的值。在某一个 commit 中，**添加了一个 bug：sum 的返回值被改成了 a + b - 1**。

`index.test.js` 是一个 sum 方法的测试用例，输入 (1, 2)，预期输出为 3

运行下列命令

```sh
git bisect start HEAD e1bc7fb7

git bisect run npx vitest run index.test.js

# f997e8a9dbf3b984f089926f02c89601293ca7c1 is the first bad commit
# commit f997e8a9dbf3b984f089926f02c89601293ca7c1
# Author: elvis liao <1219585136@qq.com>
# Date:   Mon Oct 28 15:12:18 2024 +0800
# 
#     feat: bug change
# 
#  index.js | 2 +-
#  1 file changed, 1 insertion(+), 1 deletion(-)
# bisect found first bad commit
```

可以看到输出结果中找出了导致 bug 的 `feat: bug change` commit。

1、`git bisect start` 命令可以通过后面的参数直接指定 bad commit 和 good commit：

```sh
git bisect start [bad commit] [good commit]
```

2、通过 `git bisect run [command]` 运行命令检查当前 commit 是否有问题，command 可以执行 shell 脚本，vitest 或 jest 等测试用例，只需要遵守一个约定：正常运行的时候返回 0 状态码。

运行错误的状态码细节可以在 `git bisect run --help` 或 [官方文档](https://git-scm.com/docs/git-bisect) 中查看

## 注意

使用 `git bisect run` 的时候，需要注意：由于 git bisect 是将代码回退到历史 commit 然后执行命令判断 commit 是否有问题，如果你在项目目录添加了新的一个测试用例用来找这个 bug，这个测试用例可能在回退到历史 commit 时丢失，导致意料之外的错误，影响 git bisect 的结果。为了避免这种情况，可以在项目目录之外创建测试用例来运行。

## 相关内容

> [git bisect 文档](https://git-scm.com/docs/git-bisect)

> [Debugging Till Dawn: How Git Bisect Saved My Demo](https://www.mikebuss.com/posts/debugging-till-dawn)
