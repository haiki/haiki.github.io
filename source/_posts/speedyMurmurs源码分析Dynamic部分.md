---
title: speedyMurmurs源码分析Dynamic部分
date: 2019-10-16 09:27:00
categories:
- 论文相关总结
tags:
- speedymurmurs
- payment routing
- 路由算法
---

Dynamic.class

```java
"SKIP_EXISTING_DATA_FOLDERS", "false"
"MAIN_DATA_FOLDER","./data/"
String path = "../data/";
@param args
	0: run //使用第几组数据
	1: config (0- SilentWhispers, 7- SpeedyMurmurs, 10-MaxFlow)
	2: steps previously completed  
if(step==0){
	graph =  path+"finalSets/dynamic/jan2013-lcc-t0.graph";//Number of Nodes:93502,Number of Edges: 331096
	trans = path+"finalSets/dynamic/jan2013-trans-lcc-noself-uniq-1.txt"; <四元组>
	newlinks = path+"finalSets/dynamic/jan2013-newlinks-lcc-sorted-uniq-t0.txt";<四元组>
}
//对于SW，参数分别表示：图数据，name，交易，新边，类型（SW/SM/Max），第几组数据
case 0: runDynSWSM(new String[] {graph, "SW-P"+(step+1),trans ,  newlinks, "0", ""+run}); break;
//对于SM
case 7: runDynSWSM(new String[] {graph, "SM-P"+(step+1), trans ,  newlinks, "7", ""+run}); break;
//对于MaxFlow
case 10: runMaxFlow(graph, trans, "M-P"+(step+1), newlinks, 165.55245497208898*1000); break;
```

```java
runDynSWSM(String[] args)
"SERIES_GRAPH_WRITE", ""+true
"SKIP_EXISTING_DATA_FOLDERS", "false"
graph,name,trans,add,type,i
epoch = 165.55245497208898*1000;
max = 1;//一笔交易的尝试次数就是1次
dyn,multi//对于SW，分别是false，true，对于SM，分别是true，false
	ra = new TreerouteSilentW();
		ParameterList.key = "TREE_ROUTE_TDRAP"
		ParameterList.parameters = new Parameter[0];
    	Treeroute.rand = new Random();
	ra = new TreerouteTDRAP();
		ParameterList.key = "TREE_ROUTE_SILENTW"
         ParameterList.parameters = new Parameter[0];
		Treeroute.rand = new Random();
double req = 165.55245497208898*2;
int[] roots = {64,36,43};//直接给定下来了。。。
Partitioner part = new RandomPartitioner();
	Partitioner.name = "RANDOM_PARTITIONER";
	RandomPartitioner.rand = new Random();
Network net = new ReadableFile(name, name,graph,null);
```

```java
//网络读取数据的参数
//name = "SW-P"+(step+1)/"SM-P"+(step+1)/"M-P"+(step+1)
Network net = new ReadableFile(name, name,graph,null);
	ReadableFile.filename = graph;
	ParameterList.key = "READABLE_FILE_" + name;
	ParameterList.parameters = {["NODES":93502]};
	Network.node = 93502;
	Network.transformations = new Transformation[0];
```

```java
//具体的ra算法
CreditNetwork cred = new CreditNetwork(trans, name, epoch, ra, dyn, multi, req, part, roots, max, add);	
public CreditNetwork(String file, String name, double epoch, Treeroute ra, boolean dynRep, 
			boolean multi, double requeueInt, Partitioner part, int[] roots, int max, String links){
		this(file,name,epoch,ra,dynRep, multi, requeueInt, part, roots, max, links, true);
	}
		ParameterList.key = "CREDIT_NETWORK"；
		ParameterList.parameters = new Parameter[]{new StringParameter("NAME", name), 
				new DoubleParameter("EPOCH", epoch),
				new StringParameter("RA", ra.getKey()), 
				new BooleanParameter("DYN_REPAIR", dynRep), 
				new BooleanParameter("MULTI", multi), 
				new IntParameter("TREES", roots.length),
				new DoubleParameter("REQUEUE_INTERVAL", requeueInt), 
				new StringParameter("PARTITIONER", part.getName()),
				new IntParameter("MAX_TRIES",max)});
		CreditNetwork.epoch = epoch;
		CreditNetwork.ra = ra;
		CreditNetwork.multi = multi;
		CreditNetwork.dynRepair = dynRep;
		transactions = this.readList(file);
		CreditNetwork.requeueInt = requeueInt;
		CreditNetwork.part = part;
		CreditNetwork.roots = roots;
		CreditNetwork.maxTries = max;
		if (links != null){
			this.newLinks = this.readLinks(links);	//把新边信息存起来
```

