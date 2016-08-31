---
title: CoffeeScript实现BF解释器
date: 2012-07-08
category: others
toc: false
tags:
- coffeescript
---

今天一大早天气很好，翻了几页《Go语言·云动力》，1.5节讲到下面的脑操编程语言

> ++++++++++[>++++++++++<-]>++++.+.

额，这不是大名鼎鼎的BF么？终于明白上面的"脑操"不是印刷错误，只是这翻译真不给力，感觉用"脑残"会合适点。(PS: 用BF来讲述图灵机似乎不太合适，虽然BF里有存储/跳转/输出，但是缺少了外部输入，唯一的输入就是代码本身。)

虽然之前知道BF解释器很简单，但毕竟从没亲手写过。闲来O疼便打算写个，练习下CoffeeScript也熟悉下github。还好一个小时就测好了，不到30行。为了行数更少，合并了一些赋值语句，可能是个人水平问题，很难再短了。

关于BrainFuck的介绍可以查看这个网页，这里就不多说了，实在是，很坑爹。

``` coffeescript
exports.run = (source) ->
  throw new Error 'invalid source' unless typeof(source) is 'string'
  throw new Error "invalid char '#{m[0]}'" if (m=/[^+-.<>[\]]/.exec source)?
  [i,buf,jmp] = [0,[],[]]
  for ch,p in source when ch is '[' or ch is ']'
    switch ch
      when '[' then buf[i++]=p
      when ']'
        if buf[i-1]?
          jmp[buf[--i]]=p
          jmp[p]=buf[i]
          delete buf[i]
        else
          throw new Error "unexpect ']' at pos #{p}"
  unless i is 0
    throw new Error "expect ']' to match '[' at pos #{buf[i-1]}"
  [p,len]=[-1,source.length]
  while ++p<len
    ch = source[p]
    switch ch
      when '+' then buf[i]=(buf[i] ? 0)+1
      when '-' then buf[i]=(buf[i] ? 0)-1
      when '>' then ++i
      when '<' then --i
      when '.' then process.stdout.write String.fromCharCode buf[i] ? 0x30
      when '[' then p=jmp[p] if not buf[i]? or buf[i] is 0
      when ']' then p=jmp[p] if buf[i]? and buf[i] isnt 0

exports.run "++++++++++[>++++++++++<-]>++++.+."  # print "hi"
```

然后有个问题：假设已知某种BF解释器对7种指令的时间消耗分别为T1~T7，设计一个算法，计算该解释器输出给定字符串所需的最优源代码（时间消耗最低，代码最少）