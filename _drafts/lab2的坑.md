# Fault Tolerant

# Lab2的坑
按照Lab要求完成后基本能跑过BasicFail，有点难的在于duplicate request的处理和primary和backup之间的通信（put之后p向b备份；新b向p要之前的数据）

之后。。就要自己琢磨了

## 坑一
不要把go的指针想得太智能，还是有区别的。比如我的client一直收不到put reply，即primary server已经完成put了但是client那边收到的reply一直是空。因为那个test case刚好是unreliable connection，让我一开始觉得是unreliable的问题……但是也就30%报错的概率，怎么就一直停着呢……

后来发现是指针的坑，如果这么写：

```
// reply *PutReply
if r, ok := pb.putid[args.id]; ok {
	reply = &r
	fmt.printf("duplicate put request!\n %s, %v\n", reply.err, 		reply.previousvalue)
	return nil
}
```
就会遇坑了，fmt的输出很正常，rpc那一头的reply就啥都没有了。
问题就在

```
	reply = &r
```

需要换成这样：（和C/C++很像）

```
	*reply = r
```

## 卡在Put immediate kill primary
需要Put fail到一定次数后询问新的primary，然后要注意的是`SetNewBackup`不应该写在`StartServer`里而是`tick`中，这样可以保证从idle(viewservice记录的除primary和backup外多余的server)提升到backup的server能被正常初始化。