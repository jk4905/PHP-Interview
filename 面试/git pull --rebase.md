### git pull --rebase

git pull          = git fetch + git merge
git pull --rebase = git fetch + git rebase

现在来看看git merge和git rebase的区别。

![](https://kagami-1259053372.cos.ap-chengdu.myqcloud.com/images/15722741383374.jpg)


假设有3次提交A,B,C。
在远程分支origin的基础上创建一个名为"mywork"的分支并提交了，同时有其他人在"origin"上做了一些修改并提交了。
其实这个时候E不应该提交，因为提交后会发生冲突。如何解决这些冲突呢？有以下两种方法：
![](https://kagami-1259053372.cos.ap-chengdu.myqcloud.com/images/15722741444547.jpg)



1、git merge
用git pull命令把"origin"分支上的修改pull下来与本地提交合并（merge）成版本M，但这样会形成图中的菱形，让人很困惑。
![](https://kagami-1259053372.cos.ap-chengdu.myqcloud.com/images/15722741503107.jpg)


2、git rebase
创建一个新的提交R，R的文件内容和上面M的一样，但我们将E提交废除，当它不存在（图中用虚线表示）。由于这种删除，小李不应该push其他的repository.rebase的好处是避免了菱形的产生，保持提交曲线为直线，让大家易于理解。
![](https://kagami-1259053372.cos.ap-chengdu.myqcloud.com/images/15722741564972.jpg)


区别：
1. merge 遇到冲突后就不会进行下去了，手动修改冲突后，add 修改，commit 就可以了。
2. rebase 操作的话，会终端 rebase，同时提示解决冲突。解决冲突后，将 add 修改后，执行 git rebase -continue 继续操作，或者 git rebase -skip 忽略冲突。


参考：
[git pull和git pull --rebase的不同](https://blog.csdn.net/williamfan21c/article/details/54077003)
