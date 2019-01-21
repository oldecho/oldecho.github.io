---
title: DeepgreenDB segment 节点异常恢复
date: 2019-01-21 07:07:07
categories:
- Greenplum
tags:
- Deepgreen
- segment异常
---

deepgreendb 版本: deepgreendb.18.07.180718

原文链接：http://blog.51cto.com/michaelkang/2166368

# DeepgreenDB segment 节点异常恢复

## Segment检测及故障切换机制

GP Master首先会检测Primary状态，如果Primary不可连通，那么将会检测Mirror状态，Primary/Mirror状态总共有4种：

1. Primary活着，Mirror活着。GP Master探测Primary成功之后直接返回，进行下一个Segment检测；
2. Primary活着，Mirror挂了。GP Master探测Primary成功之后，通过Primary返回的状态得知Mirror挂掉了（Mirror挂掉之后，Primary将会探测到，将自己变成ChangeTracking模式），这时候更新Master元信息，进行下一个Segment检测；
3. Primary挂了，Mirror活着。GP Master探测Primary失败之后探测Mirror，发现Mirror是活着，这时候更新Master上面的元信息，同时使Mirror接管Primary（故障切换），进行下一个Segment检测；
4. Primary挂了，Mirror挂了。GP Master探测Primary失败之后探测Mirror，Mirror也是挂了，直到重试最大值，结束这个Segment的探测，也不更新Master元信息了，进行下一个Segment检测。

上面的2-4需要进行gprecoverseg进行segment恢复。

对失败的segment节点，启动时会直接跳过，忽略。 

**所以直接重启集群没有用**

**查看集群状态** `gpstate`

**查看 mirror 节点启动状态** `gpstate -m`

## 恢复 segment

- 生成恢复配置文件 `gprecoverseg -o ./recov`

- 使用文件恢复 `gprecoverseg -i ./recov`
- primary mirror 角色对调 `gprecoverseg -r`

