# Aniverse（星大）语 录

## 关于 U2 限速问题

先上结论：U2 peer list 里显示的瞬时速度超 50MB/s 则【那一次】汇报的上传和下载流量量都不算到你的账号里

U2 某个种子的 peer list 中某个用户的【平均】上传速度 = 该用户这次连上这个种子的总上传流量除以该用户这一次连接这个种子的时间（暂停过就算中断，重新算） 比如说，这个种子我昨晚加上的，到今晚刚好24小时，在 peer list中显示我上传了1TB，那我的平均速度=1TB÷24小时=12.136MB/s 换个例子：总共跑了1000米，跑了3分钟，你的平均速度是5.55米/秒 这个速度，我一般把它叫做总体的平均速度

U2 某个种子的 peer list 中某个用户的【瞬时】上传速度 = 该用户这次汇报 Tracker 时的上传量增量除以两次汇报 Tracker 的时间间隔 比如说第三次你汇报 Tracker 时你的连接时间是3个小时，上传量是500GB，过了20分钟后你又汇报了 Tracker，汇报时的你的上传量是560GB，那你这一次汇报时候的平均速度=\(560GB-500GB\)÷\(20分钟\)=51.20MB/s 于是你会在 peer list 看到这么个数据： 上传量=560GB，平均\(上传\)速度=47.786MB/s，瞬时\(上传\)速度=51.200MB/s，连接时间=3:20:00 换个例子：你总共要跑1200米，现在跑了450米，你在通过第一个检查点的时候距离开跑的时间是20秒，你跑过的距离是190米；而你到第二个检查点的时候距离开跑已经过了45秒，你跑过的距离是370米，那你在这两个检查点之间的平均速度是\(370-190\)÷\(45-20\)=7.2米/秒。而这个平均速度在U2被称之为瞬时速度 这个速度我更愿意把它叫做单次汇报 Tracker 时的平均速度

那么什么是真正的瞬时速度？ 相对来说你在客户端里看到的速度更接近于实际的瞬时速度（但其实也不是真正的瞬时速度，客户端显示的速度其实也不准确） 此外就是，PT 站是不知道你真正的瞬时速度的，他对于你速度的计算就是靠多次汇报 Tracker 时上传量的增量除以相隔时间来计算

换言之，如果你要在U2不超速，只要你30分钟内汇报的上传量增量不超过 87.890 GB 即可，你在这30分钟内跑出了 1GB/s 甚至 10GB/s 都没事，只要最后这30分钟内的增量不大于 87.890 GB，那你的“瞬时速度”（U2 peer list 里的瞬时速度）就不会大于 50MB/s

## 关于libtorrent-rasterbar兼容性问题

1. 确实，Deluge 1.3.15 与 libtorrent-rasterbar 1.1.x 分支的兼容性还不是很好，比如你们知道的 WebUI 连不上（其实是连不上守护进程）就和 1.1.x 有关。这个问题我一直知道
2. 我脚本默认选项是 1.1.x 还是 1.0.x 和你对 qBittorrent 和 Deluge 的选择有关，当 Deluge 版本选择 2.0 及以上，或者 qBittorrent 版本选择 4.1.4 及以上的时候，libtorrent 默认选项就会变为 1.1.x，且无法选择 1.0.x（除非你手动指定）。除了我说的这个情况以外，其他时候 libtorrent 的默认选项都是 1.0.x。不过由于脚本默认选择安装 qb 4.1.5，所以 libtorrent 变相的默认就是 1.1.x
3. 下面我来说说下我这么做的原因

* libtorrent-rasterbar（后简称 lt） 是 Deluge（后简称 de） 和 qBittorrent（后简称 qb） 的后端，由于镜像里带的版本都不怎么好用甚至就不能用（作为刷子我觉得不满意，包括严重 bug、性能问题，汇报问题等等），我选择自己编译
* 自己编译的话，de 和 qb 的编译参数有所不同，从省时间的角度来说我偏向于编译一次，这样的话 de 和 qb 都是用这个编译的 lt
* qb 我推荐使用最新的 4.1.5。4.1.5 没有明确说抛弃了对 lt 1.0 的支持，但是在实际编译过程中我测试多次都无法成功编译，换了 1.1 分支就能编译出来。作为一只菜鸡我不知何故，也看不懂代码，我就当做 qb 4.1.5 无法支持 lt 1.0 吧（实际上从 qb 4.1.4 开始就编译不出来了，qb 4.1.3 还是可以的）
* lt 1.1 我以前不用的原因是，在 de/qb 上有各种 bug，比如限速限不住、IPv6 汇报有问题、port 0 的误报等等——但近来这些 bug 都已被解决，并且 lt 1.1 在磁盘性能上有不小的提升，因此在不考虑出其他 bug 的情况下我优先考虑使用 lt 1.1
* 现在的情况是 ，qb 4.1.5 必须要用 lt 1.1，de 1.3.15 对 lt 1.1 有更容易崩溃和高负载状态下容易连不上守护进程的 bug。我认为 de 使用 lt 1.1 带来的弊端是在可接受的范围内，而 lt 1.1 对 qb 4.1.5 的必要性以及性能提升这些优点对我来说更重要，因此综合利弊后我还是决定在 de 上也使用 lt 1.1
* 我确实也可以分开编译，让 de 使用 lt 1.0，qb 使用 1.1。不愿意这么做的原因主要是考虑到编译 lt 比较慢的问题，以及……绝大多数人的盒子还是 HDD 盒子，就我得到的反馈来看，lt 1.1 在 HDD 盒子上带来的磁盘性能提升还是比较大的，我认为即便有些严重 bug 还是有使用 1.1 分支的价值的

1. 如果你不需要 qb 4.1.4 及以后的版本，你可以在脚本里选择不装 qb 或者就装 4.1.3 及以前的版本，然后在脚本里选择 lt 1.0.11
2. 如果你需要 qb 4.1.4 以及之后的版本，也想用 de 1.3 + lt 1.0 的话，我脚本不提供解决方法，你可以看 de 官方给出的编译教程来自行编译
3. 告诉我为什么要用 1.0.9？就因为 CentOS 某个源里带的 de 附带的 lt 版本是 1.0.9？1.0.11 总归比 1.0.9 好吧。并且我即便用 1.0.11 我也不用 release 版，我用 RC\_1\_0 分支编译的（有些 backport 加上去以后没做 1.0.12 的 release）

我再来说点碎碎念、个人想法。  
qb 4.2.0 要抛弃 lt 1.1.11 以前的版本了  
de 2.0 也早已经确定要抛弃 lt 1.0 的支持了，然而 de stable 已经很久没更新了，dev 分支也很少有更新，有生之年系列的感觉。几个月前我装过最新的 dev 分支，一些新 feature 我觉得挺有意思，然而能感受到的 bug 也不少。现阶段也几乎没有站点能支持 2.0 dev，因此用 de 2.0 dev 配合 lt 1.1 也不太现实

我感受到 de 用了 lt 1.1 后，崩溃的概率是比 1.0 要高，不过这个频率不算高，我也没碰到过几次——况且我的 systemd 里也有自动重启的设定，虽然重启还是会有红种、fast resume 信息丢失等损失发生的可能性，但我觉得还在可接受范围内  
至于高负载（和速度有关但不能说完全是速度的原因，我跑 300MB/s 也不见得都失联啊）下连不上守护进程的问题，这个我也比较烦恼。连不上的是守护进程，因此不仅 WebUI，thin client 都连不上，一定要用的话只能看 SSH。这确实是一个大问题，如果很在乎这个问题的话，还是使用 1.0 吧

