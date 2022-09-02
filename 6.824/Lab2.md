## 函数

### Make(peers,me,…)

`Make(peers,me,…)` to create a Raft peer. The peers argument is an array of network identifiers of the Raft peers (including this one), for use with RPC. The `me` argument is the index of this peer in the peers array.

`作用：create a new Raft server instance`

创建一个Raft peer，参数peers是一个网络标识符的数组关于(the Raft peers)，用作RPC 。参数me是该peer在数组中的索引。

### Start((command interface{}) (index, term, isleader))

`Start(command)` asks Raft to start the processing to append the command to the replicated log. `Start()` should return immediately, without waiting for the log appends to complete. The service expects your implementation to send an `ApplyMsg` for each newly committed log entry to the `applyCh` channel argument to `Make()`.

```
参数：command   interface类型

返回值：int index, int term, bool isleader

作用：start agreement on a new log entry
```

要求Raft开始进行添加command到the replicated log，该函数应该马上返回，不需要等待添加操作完成。这个服务要求你的实现去为每一个新commit的log entry发送ApplyMsg到Make()的`applyCh`（applyCh是Make()的一个参数）。

### GetState()

```
 作用：ask a Raft for its current term, and whether it thinks it is leader
```



## 结构体

### ApplyMsg

```
// each time a new entry is committed to the log, each Raft peer
// should send an ApplyMsg to the service (or tester).
```

```
每次一个新的entry被committed到log时，每一个Raft peer应该发送一个ApplyMSg给service或者tester
```

## Part2A leader election 

```
Implement Raft leader election and heartbeats (AppendEntries RPCs with no log entries). The goal for Part 2A is for a single leader to be elected, for the leader to remain the leader if there are no failures, and for a new leader to take over if the old leader fails or if packets to/from the old leader are lost. Run go test -run 2A to test your 2A code. 
```

实现leader选举和心跳。

目标：单个leader选举，当没有错误发生，leader不变。当旧的leader失败或者旧的leader发送或者接受的packet丢失了，选出新的leader来替代。

### 提示：

1. 通过`go test -run 2A`来测试。
2. 按照论文中图2。关心发送和接收`RequestVote RPCs`，Server关于选举的规则，以及关于leader选举the State。
3. 将图 2 中的leader选举状态添加到 `raft.go` 中的 `Raft `结构中。也需要定义一个结构来保存有关每个`log entry`的信息。
3. 补充`RequestVoteArgs` 和 `RequestVoteReply` 结构。  修改 `Make()` 以创建一个后台 `goroutine`，该 `goroutine` 如果一段时间内没有收到其他peer的消息，将通过发送`RequestVote`Rpcs来周期性地开始leader选举。  这样，peer将了解谁是领导者，这里是否已经有领导者，或者自己成为领导者。实现 RequestVote() RPC 处理程序，以便服务器相互投票。
3. 要实现心跳，请定义一个 AppendEntries RPC 结构（尽管您可能还不需要所有参数），并让领导者定期发送它们。  编写一个 AppendEntries RPC 处理程序方法来重置选举超时，这样Server就不会在一个Server已经被选举为leader的时候作为leader站出来。
3. 确保不同节点的选举超时不会总是同时触发，否则所有节点只会为自己投票，没有人会成为领导者。
3. 测试者要求leader每秒发送心跳RPC不超过十次。
3. 测试人员要求您的 Raft 在旧领导者失败后的 5 秒内选举新领导者（如果majoritiy of peer仍然可以通信）。但是请记住，在分裂投票的情况下，领导者选举可能需要多轮（如果数据包丢失或候选人不幸选择了相同的随机退避时间，则可能发生这种情况）。 您必须选择足够短的选举超时（以及心跳间隔），以使选举很可能在不到五秒的时间内完成，即使它需要多轮。
3. 该论文的第 5.2 节提到了 150 到 300 毫秒范围内的选举超时。  只有当领导者发送心跳的频率大大高于每 150 毫秒一次时，这样的范围才有意义。  由于测试人员将您限制为每秒 10 次心跳，您将不得不使用大于论文中 150 到 300 毫秒的选举超时，但不能太大，因为那样您可能无法在 5 秒内选举领导者。
3. 你可能会发现 Go 的 rand 很有用
3. 您需要编写定期或在延迟时间后采取行动的代码。  最简单的方法是创建一个带有调用 time.Sleep() 的循环的 goroutine；  （参见 Make() 为此目的创建的ticker() goroutine）。  不要使用 Go 的 time.Timer 或 time.Ticker，它们很难正确使用。
3. 指导页面有一些关于如何开发和调试代码的提示
3. 如果您的代码无法通过测试，请再次阅读论文的图 2；  领导者选举的完整逻辑分布在图中的多个部分。
3. 不要忘记实现 GetState()。
3. 测试人员在永久关闭实例时调用 Raft 的 rf.Kill()。  您可以使用 rf.killed() 检查是否已调用 Kill()。  您可能希望在所有循环中都这样做，以避免死的 Raft 实例打印出令人困惑的消息。
3. Go RPC 仅发送名称以大写字母开头的结构字段。  子结构还必须具有大写的字段名称（例如，数组中的日志记录字段）。  labgob 包会警告你这一点；  不要忽视警告。



1. You can't easily run your Raft implementation directly; instead you should run it by way of the tester, i.e. `go test -run 2A `.

2. Follow the paper's Figure 2. At this point you care about sending and receiving RequestVote RPCs, the Rules for Servers that relate to elections, and the State related to leader election,

3. Add the Figure 2 state for leader election to the `Raft` struct in `raft.go`. You'll also need to define a struct to hold information about each log entry.

4. Fill in the `RequestVoteArgs` and `RequestVoteReply` structs. Modify `Make()` to create a background goroutine that will kick off leader election periodically by sending out `RequestVote` RPCs when it hasn't heard from another peer for a while.  This way a peer will learn who is the leader, if there is already a leader, or become the leader itself.  Implement the `RequestVote()` RPC handler so that servers will vote for one another.

5. To implement heartbeats, define an `AppendEntries` RPC struct (though you may not need all the arguments yet), and have the leader send them out periodically. Write an `AppendEntries` RPC handler method that resets the election timeout so that other servers don't step forward as leaders when one has already been elected.

6. Make sure the election timeouts in different peers don't always fire at the same time, or else all peers will vote only for themselves and no one will become the leader.

7. The tester requires that the leader send heartbeat RPCs no more than ten times per second.

8. The tester requires your Raft to elect a new leader within five seconds of the failure of the old leader (if a majority of peers can still communicate). Remember, however, that leader election may require multiple rounds in case of a split vote (which can happen if packets are lost or if candidates unluckily choose the same random backoff times). You must pick election timeouts (and thus heartbeat intervals) that are short enough that it's very likely that an election will complete in less than five seconds even if it requires multiple rounds.

9. The paper's Section 5.2 mentions election timeouts in the range of 150 to 300 milliseconds. Such a range only makes sense if the leader sends heartbeats considerably more often than once per 150 milliseconds. Because the tester limits you to 10 heartbeats per second, you will have to use an election timeout larger than the paper's 150 to 300 milliseconds, but not too large, because then you may fail to elect a leader within five seconds.

10. You may find Go's [rand](https://golang.org/pkg/math/rand/) useful.

11. You'll need to write code that takes actions periodically or after delays in time. The easiest way to do this is to create a goroutine with a loop that calls [time.Sleep()](https://golang.org/pkg/time/#Sleep); (see the `ticker()` goroutine that `Make()` creates for this purpose). Don't use Go's `time.Timer` or `time.Ticker`, which are difficult to use correctly.

12. The [Guidance page](https://pdos.csail.mit.edu/6.824/labs/guidance.html) has some  tips on how to develop and debug your code.

13. If your code has trouble passing the tests, read the paper's Figure 2 again; the full logic for leader election is spread over multiple parts of the figure.

14. Don't forget to implement `GetState()`.

15. The tester calls your Raft's `rf.Kill()` when it is permanently shutting down an instance. You can check whether `Kill()` has been called using `rf.killed()`. You may want to do this in all loops, to avoid having dead Raft instances print confusing messages.

16. Go RPC sends only struct fields whose names start with capital letters.  Sub-structures must also have capitalized field names (e.g. fields of log records  in an array). The `labgob` package will warn you about this;  don't ignore the warnings.