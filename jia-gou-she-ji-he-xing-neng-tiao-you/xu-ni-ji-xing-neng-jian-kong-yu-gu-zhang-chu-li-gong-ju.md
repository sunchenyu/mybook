# 虚拟机性能监控与故障处理工具

## jps1

列出正在运行的虚拟机进程，列出虚拟机执行主类，名称，以及这些进程的本地虚拟机唯一ID



jps -l 输出进程pid 和 主类的全限定名或JAR 的完整路径

<div align="left"><figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure></div>

jps -m 输出传递给主类方法的参数

<div align="left"><img src="../.gitbook/assets/0 (1) (1)" alt="0" height="200" width="1454"></div>

jps -v 输出虚拟机进程启动时的jvm参数

<div align="left"><img src="../.gitbook/assets/0 (2)" alt="0" height="285" width="1662"></div>

jps -q 只输出进程pid

<div align="left"><img src="../.gitbook/assets/0 (3)" alt="0" height="196" width="246"></div>

## jstat

j监视虚拟机各种运行状态信息的命令行工具



jstat -gc vmid 监视jvm堆状况，包括Eden区、两个suvivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息

<table data-header-hidden><thead><tr><th width="99"></th><th width="229"></th><th width="298"></th><th></th></tr></thead><tbody><tr><td>字段</td><td>全名</td><td>含义</td><td>单位</td></tr><tr><td>S0C</td><td>Survivor 0 Capacity</td><td>Survivor 区 S0 的总容量</td><td>KB</td></tr><tr><td>S1C</td><td>Survivor 1 Capacity</td><td>Survivor 区 S1 的总容量</td><td>KB</td></tr><tr><td>S0U</td><td>Survivor 0 Used</td><td>Survivor S0 当前使用量</td><td>KB</td></tr><tr><td>S1U</td><td>Survivor 1 Used</td><td>Survivor S1 当前使用量</td><td>KB</td></tr><tr><td>EC</td><td>Eden Capacity</td><td>Eden 区容量</td><td>KB</td></tr><tr><td>EU</td><td>Eden Used</td><td>Eden 区已使用</td><td>KB</td></tr><tr><td>OC</td><td>Old Capacity</td><td>Old（老年代）容量</td><td>KB</td></tr><tr><td>OU</td><td>Old Used</td><td>Old（老年代）已使用</td><td>KB</td></tr><tr><td>MC</td><td>Metaspace Capacity</td><td>元空间容量（可能是 committed 大小）</td><td>KB</td></tr><tr><td>MU</td><td>Metaspace Used</td><td>元空间已使用</td><td>KB</td></tr><tr><td>CCSC</td><td>Compressed Class Space Capacity</td><td>Compressed Class Space 的容量</td><td>KB</td></tr><tr><td>CCSU</td><td>Compressed Class Space Used</td><td>Compressed Class Space 已使用</td><td>KB</td></tr><tr><td>YGC</td><td>Young GC Count</td><td>发生 Young GC 的次数（次）</td><td>次</td></tr><tr><td>YGCT</td><td>Young GC Time</td><td>Young GC 总耗时（秒）</td><td>s</td></tr><tr><td>FGC</td><td>Full GC Count</td><td>Full GC 的次数</td><td>次</td></tr><tr><td>FGCT</td><td>Full GC Time</td><td>Full GC 总耗时（秒）</td><td>s</td></tr><tr><td>GCT</td><td>GC Total Time</td><td>GC（Young+Full）所有耗时总和（秒）</td><td>s</td></tr></tbody></table>

<figure><img src="../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

jinfo java配置信息工具

实时查看并修改虚拟机各项参数

<div align="left"><figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure></div>

jmap 内存映像工具，用于生成堆转储快照

jmap -heap 15232

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure></div>



显示堆中对象统计信息，包括类、实例数量、合计容量

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure></div>



jhat 堆转储快照分析工具





jstack堆栈跟踪工具





参考《深入理解java虚拟机》
