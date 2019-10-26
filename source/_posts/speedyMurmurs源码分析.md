---
title: speedyMurmurs源码分析
date: 2019-10-11 20:12:57
categories:
- 论文相关总结
tags: 
- speedymurmurs
- payment routing
- 路由算法
---

Static.main



### Config文件的读取

```java
String defaultConfigFolder = "./config/";

Config.overwrite(String key, String value) <speedy\src\gtna\util\Config.java>
	if properties == null  // Properties extends Hashtable<Object,Object>
		Config.init() //将./config中的所有.properties文件内容加载到properties中；先得到./config下的文件夹绝对路径
			Config.initWithFolders	//然后得到各文件夹下的.properties文件绝对路径
				Config.initWithFiles //读取.properties文件	
					Config.addFile	
				Util.toStringArray(v)	//用来把存储在vector中的文件绝对路径转换成字符数组string[]
	overwrite.put(key,val);// HashMap<String, String> overwrite;
```



### 参数

```java
@param args
 0: run (integer 0-19)   //对应于20组交易数据，
 1: config (0: LM-MUL-PER(SilentWhispers), 1: LM-RAND-PER, 2: LM-MUL-OND, 3:LM-RAND-OND, 4: GE-MUL-PER, 5: GE-RAND-PER, 6: GE-MUL-OND, 7:GE-RAND-OND (SpeedyMurmurs), 8: ONLY-MUL-PER, 9:ONLY-RAND-OND, 10: max flow) 
 2: #transaction attempts             //交易重新发送次数，跑的时候使用了2；对比实验是1-10
 3: #embeddings/trees (integer > 0)         //用了3；对比实验是1-7

transList = "../data/finalSets/static/sampleTr-" + i + ".txt"; <src,dst,val,time>
graph = "../data/finalSets/static/ripple-lcc.graph";//<节点：相邻节点1；相邻节点2...相邻节点n>
degFile = "../data/finalSets/static/degOrder-bi.txt";//是排序的，每一行的数代表对应行度数的节点index<度数为i的节点>
name = "STATIC";	//实验名称
epoch = 1000;	//每1000笔交易算一个epoch
tl = 2 * epoch;	//重试交易的时间间隔
up = false; //无update
```

```java
//为不同树上的路径分配路由的资金数
Partitioner part = new RandomPartitioner();	//实例化Partitioner
	RandomPartitioner.rand =  new Random();
	Partitioner.name = "RANDOM_PARTITIONER";
```

```java
// 选择生成树的根节点
// 其中，random表示随机选取root节点，当为false时，使用最大度的方法，即在degFile中选择前trees个节点。
//如果使用随机，则使用i（当前是第几个run）作为随机种子初始化rand，之后遍历文件得到最大度，以最大度为界，随机出root（rand.nextInt）
Misc.selectRoots(String file, boolean random, int trees, int seed)	
	Misc.selectRoots(degFile, random:false, trees, i)
```



```java
//实例化不同的路由算法RA
Treeroute sW = new TreerouteSilentW();
	Metric.key = "TREE_ROUTE_SILENTW";
	Metric.parameters = new Parameter[0];
	Treeroute.rand = new Random();
	
Treeroute voute = new TreerouteTDRAP();//TreerouteTDRAP继承TreerouteNH，TreerouteNH继承Treeroute
	TreerouteNH
		Metric.key = "TREE_ROUTE_TDRAP";
		Metric.parameters = new Parameter[0];
		Treeroute.rand = new Random();
			
Treeroute only = new TreerouteOnly();
	Metric.key = "TREE_ROUTE_ONLY";
	Metric.parameters = new Parameter[0];
	Treeroute.rand = new Random();
```

```java
// 各种路由的参数组合实例化
//vary dynRepair, multi, routing algo -> 8 poss + 2 treeonly versions
// String[] com = { "SW-PER-MUL", "SW-PER", "SW-DYN-MUL", "SW-DYN", "V-PER-MUL", "V-PER", "V-DYN-MUL", "V-DYN", "TREE-ONLY1", "TREE-ONLY1" };
//三种不同的路由算法，使用不同的组合，实例化CreditNetwork	
CreditNetwork silentW = new CreditNetwork(transList, name, epoch, sW, false, true, tl, part, roots, tries, up); 
	public CreditNetwork(String file, String name, double epoch, Treeroute ra, boolean dynRep, 
			boolean multi, double requeueInt, Partitioner part, int[] roots, int max, boolean up){
		this(file,name,epoch,ra,dynRep, multi, requeueInt, part, roots, max, null,up);//调用本类中的重载函数
	}	
		public CreditNetwork(String file, String name, double epoch, Treeroute ra, boolean dynRep, 
				boolean multi, double requeueInt, Partitioner part, int[] roots, int max, String links, boolean up)
			Metric
				ParameterList.key = "CREDIT_NETWORK"；
				ParameterList.parameters = new Parameter[]{
						new StringParameter("NAME", name), 
						new DoubleParameter("EPOCH", epoch),
						new StringParameter("RA", ra.getKey()), 
						new BooleanParameter("DYN_REPAIR", dynRep), 
						new BooleanParameter("MULTI", multi), 
						new IntParameter("TREES", roots.length),
						new DoubleParameter("REQUEUE_INTERVAL", requeueInt), 
						new StringParameter("PARTITIONER", part.getName()),
						new IntParameter("MAX_TRIES",max)});
```



```java
//network实例化，加载图数据，节点个数，文件夹
//Config.overwrite("READABLE_FILE_" + folder + "_NAME_SHORT", name); 初始化当前网络的名称
// String[] com = { "SW-PER-MUL", "SW-PER", "SW-DYN-MUL", "SW-DYN", "V-PER-MUL", "V-PER", "V-DYN-MUL", "V-DYN", "TREE-ONLY1", "TREE-ONLY1" };
Network network = new ReadableFile(com[config], com[config], graph, t:null);	
public ReadableFile(String name, String folder, String filename,Transformation[] t)	
	this(name, folder, filename, new Parameter[0], t);
		ReadableFile(String name, String folder, String filename, Parameter[] parameters, Transformation[] t)
			ParameterList.key = "READABLE_FILE_" + folder;
			ParameterList.parameters = new IntParameter("NODES", nodes);//这个nodes数是从graph中的第二行得到的67149
			Network.nodes = nodes;
			Network.transformations = transformations;//null
```



```java
//这里是整个程序真正的关键执行入口处，series的实例化
//Metric[] m = new Metric[] { silentW, silentWnoMul, silentWdyn, silentWdynNoMul, vouteMulnoDyn, voutenoDyn, vouteMul,voutenoMul, treeonly1, treeonly2 };
Series.generate(network, new Metric[] { m[config] }, i, i);
Series generate(Network nw, Metric[] metrics, int startRun,int endRun)	
	Series s = new Series(nw, metrics);
		s.network = network;
		s.metrics = metrics;
		s.mainDataFolder = Config.get("MAIN_DATA_FOLDER");//生成数据的文件夹，./data/static/，这个在config文件中是./data，但是程序运行一开始就overwrite成./data/static了
	folder.mkdirs();	//older = new File(s.getFolder());生成 ./data/static/READABLE_FILE_SW-PER-MUL-67149/，用的是network对象
	folder.mkdirs();	//folder = new File(s.getFolder(m)); 由metric对象，得到文件名称 "CREDIT_NETWORK-9个参数"，见CreditNetwork
	
	Series.generateRun(s, run);
```

```java
//创建文件夹，图数据读取
Series.generateRun(s, run);	
	System.out.println("\n" + run + ":");
	ArrayList<Single> runtimes = new ArrayList<Single>();
	ArrayList<Single> etc = new ArrayList<Single>();
	File folder = new File(s.getSeriesFolderRun(run));//这里得到的folder应该是./data/static/READABLE_XXX_/0...19
	输出：G: SW-PER-MUL(N = 67149)//使用timer输出的一段话，记录图生成时间，意指graph or generate
	Graph g = s.getNetwork().generate();//ReadableFile,生成图，ripple-lcc.graph中有节点和边的信息了，这里需要用ripple-lcc-CREDIT_LINKS.graph加载权重信息，即添加图的properties
		ReadableFile.generate();
			Graph graph = new GtnaGraphReader().readWithProperties(this.filename);//filename是graph的绝对路径，这里是读取creditlinks的权值信息
				readWithProperties(filename, properties);//GraphReader.java; properties是ripple-lcc-CREDIT_LINKS.graph的绝对路径
					GraphProperty property = (GraphProperty) ClassLoader.getSystemClassLoader().loadClass(className).newInstance();//感觉是CreditLink类的实例化
					String key = property.read(prop);// CreditLinks.read(filename);//将所有的权值信息保存在CreditLink.weights中，并返回key值CREDIT_LINKS
					graph.addProperty(key, property);//key = REDIT_LINKS，即ripple-lcc-CREDIT_LINKS.graph中的key
			graph.setName(this.getDescription());
				Graph.name = ???
	runtimes.add(new Single("G_RUNTIME", timer.getRuntime()));//./data/static/READABLE_V-ROUTE_67149/0...19/runtime.txt：G_RUNTIME=0.54
	输出：P: CREDIT_LINKS//意指 properties
	folder = new File(s.getMetricFolder(run, m));//folder.mkdirs();//CREDIT_NETWORK-STATIC-1000.0-TREE_ROUTE_SILENTW-false-true-3-2000.0-RANDOM_PARTITIONER-2
	输出：M：CNET(name=STATIC,epoch=1000,ra-Tree=XXX,dr=XX,mul=XX,trees=X,ri=XX,part=XXX,mt=XX)
	m.computeData(g, s.getNetwork(), metrics);
	runtimes.add(new Single(m.getRuntimeSingleName(), timer.getRuntime()));//runtime.txt:CREDIT_NETWORK-STATIC-1000.0-TREE_ROUTE_SILENTW-false-true-1-2000.0-RANDOM_PARTITIONER-2_RUNTIME=86.41
	m.writeData(s.getMetricFolder(run, m));//CreditNetwork.java;把生成的结果数据写入文件夹；folder是CREDIT_NETWORK-STATIC-1000.0-TREE_ROUTE_SILENTW-false-true-3-2000.0-RANDOM_PARTITIONER-2下的各个.txt文件
	SingleList singleList = new SingleList(m, m.getSingles());//this.metric = metric; this.singles = singles;this.map = new HashMap<String, Single>();
		m.getSingles()；//得到每个single
			return new Single[]{m_av, m_Re_av, m_S_av, m_F_av,p_av, p_Re_av, p_S_av, p_F_av, reL_av, ls_av, s_av, s1, s, pP_av, pPF_av, pPNF_av, rt,d1,d2,d3};
	singleList.write(s.getSinglesFilenameRun(run, m));//写入_singles.txt文件
	SingleList rt = new SingleList(null, runtimes);
	rt.write(s.getRuntimesFilenameRun(run));// runtimes.txt 文件写入
	//etc文件写入
	int mb = 1024 * 1024;
	Runtime runtime = Runtime.getRuntime();
	double used = (runtime.totalMemory() - runtime.freeMemory()) / mb;
	etc.add(new Single("MEMORY_USED", used));
	SingleList etcSl = new SingleList(null, etc);
	etcSl.write(s.getEtcFilename(run));
```

```java
//路由，执行交易，统计各项数据
m.computeData(g, s.getNetwork(), metrics);
	Treeembedding embed = new Treeembedding("T",60,roots, MultipleSpanningTree.Direct.TWOPHASE);{
		this(name,pad,roots.length,turnSelector(roots),1,false, dir);
	}	
	Treeembedding(String name, int pad, int k, String rootSelector, double p, boolean depth, MultipleSpanningTree.Direct dir)	
		ParameterList.key = "TREE_EMBEDDING";
		ParameterList.parameters = new Parameter[]{
					new StringParameter("NAME", name), //"T"
					new IntParameter("PAD", pad), 
					new IntParameter("TREES",k), 
					new StringParameter("ROOT",rootSelector),//就是把root的索引拼接起来，1-2-3这样
					new DoubleParameter("P",p), //1
					new BooleanParameter("DEPTH", depth), //false
					new StringParameter("PARENT_DIR",dir.name())}//TWOPHASE
		Transformation.times = 1;
		Treeembedding.padding = pad;
		Treeembedding.trees = k;
		rand = new Random();
		Treeembedding.rootSelector = rootSelector;
		Treeembedding.p = p;
		Treeembedding.depth = depth;
		Treeembedding.dir = dir;
	if (!g.hasProperty("SPANNINGTREE_0")){  
		  g = embed.transform(g); //给这个图生成规定的生成树，先进入Treeembedding.java，生成生成树。之后又进入spanningtree.java，给生成树的每个节点生成坐标
		}
	//读取交易列表，0: decide which is next transaction: previous one or new one? and add new links if any
	//1: check if and how many spanning tree re-construction took place since last transaction
		//do 1 (!) re-computation if there was any & set stabilization cost
		//如果不是实时动态修复，那么就是periodical的，SilentWhispers为代表，需要每过一个epoch计算生成树，和坐标,需要记录生成树的稳定开销。
		//这里对于SM是没有开销的，因为它在静态情况下是没有新边的加入，生成树是不需要变化的
		//计算开销的方法:
			for (int j = epoch_old +1; j <= epoch_cur; j++){
				stabMes.add(this.roots.length*2*this.computeNonZeroEdges(g, edgeweights));//树的个数*2*非零的边数
			}
			//有些度量的计算方法完全看不懂啊。。。。
	//2: execute the transaction，分为 Multi 和 Adhoc
	if (this.multi){
		results = this.routeMulti(cur, g, nodes, exclude, edgeweights);
	} else {
		results = this.routeAdhoc(cur, g, nodes, exclude, edgeweights);
	}
	//re-queue if necessary
	
	//3 update metrics accordingly
		path = this.inc(path, results[1]); //results[1]应该是一个比较大的数吧，在这里可以做Index？？？
		reLand = this.inc(reLand, results[2]);//receiver-land
		landSen = this.inc(landSen, results[3]);// land - sender
		mes = this.inc(mes, results[4]);
		del = this.inc(del, results[5]);
		//成功情况下
		mesAll = this.inc(mesAll, cur.mes);
		pathAll = this.inc(pathAll, cur.path);
		pathSucc = this.inc(pathSucc, results[1]);
		mesSucc = this.inc(mesSucc, results[4]);
		delSucc = this.inc(delSucc, results[5]); //delay
		
		pathSF = this.inc(pathSF, val);
		pathSsF[j-6] = this.inc(pathSsF[j-6], val);
	//4 post-processing: remove edges set to 0, update spanning tree if dynRapir
	if (this.dynRepair && zeroEdges != null) 
		// 找到zeroEdges，对于边的src，dest，在生成树中是孩子节点的那个，做为repair的subroot
		cur_stab = cur_stab + this.repairTree(nodes, sp, coords, cut, (CreditLinks) g.getProperty("CREDIT_LINKS"));//开销就是边的改动	
	if (!this.update){
				this.weightUpdate(edgeweights, originalWeight);//因为是静态的，这里就不会对原CreditLink中的值做改变
			}
	/SM是所有的交易都执行完，执行这个
		if (this.dynRepair){
			stabMes.add(cur_stab);
		}		
	//compute metrics，使用Distribution

```

```java
//生成树的生成和坐标的生成
g = embed.transform(g);
public Graph transform(Graph g)//这些参数可以直接参见Treeembedding
	Transformation tbfs = new MultipleSpanningTree(this.rootSelector,this.trees, rand,this.p,this.depth, this.dir);
		ParameterList.key = "SPANNINGTREE_BFS";
		ParameterList.parameters = new Parameter[] { 
					new StringParameter("ROOT_SELECTOR", rootSelector), 
					new BooleanParameter("RANDOM_ORDER",true), 
					new StringParameter("ONED",oneD.name())});
		Transformation.times = 1;
		MultipleSpanningTree.rootSelector = rootSelector;
		randomorder = true;
	    MultipleSpanningTree.rand = rand;
	    MultipleSpanningTree.trees = k;
	    MultipleSpanningTree.p = p;
	    MultipleSpanningTree.d = depth;
	    MultipleSpanningTree.oneDSel = oneD;//TWOPHASE
	g = tbfs.transform(g);//MultipleSpanningTree.transform(g) //多个生成树，确定根节点，孩子父亲节点
		CreditLinks: Map<Edge, double[]> weights;//graph-lcc-CREDIT_LINKS
		int[] r = this.selectRoot(graph, rootSelector);//又把1-2-3这种形式给分成了｛1，2，3｝
		//对于所有的节点
		if (this.oneDSel ==  Direct.TWOPHASE){
			l = potParents(graph, nodes[i], Direct.NONE, edgeweights).length;//得到的是当前节点的无重复的相邻节点的个数，不对边做剔除
				case NONE: return potChildren(g, n, Direct.NONE, ew);
					ew.getPot(in[j], n.getIndex()) > 0
		}
		//对于不同的生成树
		//分别得到根节点的入边，出边，筛选出容量大于0的节点，统计出与此节点双向的合法节点：即出和入相同的节点
		int[] out = potChildren(graph, nodes[roots[i]], this.oneDSel, edgeweights);// this.oneDSel ==  Direct.TWOPHASE
		
	//给生成树赋值坐标，根节点是空，其他节点的坐标是<其父节点的坐标+随机数>
		//pad coordinates
			//coordinate 是二维数组，第一维的长度就是节点数，第二维的长度取决于当前节点在生成树中的的层数
			// 对于根节点，二维长度为0，对于第一层的节点，二维长度为1...对于 padding 则是将所有的第二维数组，填充为一样长度
```

```java
/**
	 * routing using a multi-part computation: costs are
	 * i) finding path (one-way, but 3x path length as each neighbor needs to sign its predecessor and successors)
	 * ii) sending shares to all landmarks/roots from receiver 
	 * iii) sending results to sender from all landmarks/roots
	 * iv) testing paths (two-way)
	 * v) updating credit (one-way)
	 * @return {success?0:-1, sum(pathlength), receiver-landmark, landmarks-sender,overall message count, delay, p1, p2,...}
	 */	
results = this.routeMulti(cur, g, nodes, exclude, edgeweights);
int[] routeMulti(Transaction cur, Graph g, Node[] nodes, boolean[] exclude, CreditLinks edgeweights)	
	//对于不同的三棵树，在每颗树上找路径
	paths[j] = ra.getRoute(src, dest, j, g, nodes, exclude);
		initRoute(); //对于SW，置 up 为true，up是向上（到根）找路的意思；对于TreerouteOnly，无代码
		int next = this.nextHop(src, nodes,destC,dest,exclude,pre);
			//首先计算src 到 root 的路径
			index = sp.getParent(cur);
			//再计算 root 到 dst 的路径，找出最近的；父亲只有一个，但是孩子有很多个，要选最好的那个
			int[] out = sp.getChildren(cur);
			int dbest = this.getCPL(dest, this.coords[cur]);//计算src和dest坐标的相同位数，再在孩子节点中找比离 dest 更近的节点	
		//返回从 src 到 dest 的路径，有可能找不到路，即没有下一跳，此时需要判断，path的最后一个值是否为dest
	//compute the  minimum credit along the paths，计算找到的每个树的路径上的最小值
	double w = edgeweights.getPot(l,k);//对路径上的所有边，求最小
	//partition transaction value，将一笔交易要转移的val，拆分成每个路径上合适的金额
	vals = part.partition(g, src,dest,cur.val,mins);
		//首先将一个 val， 随机的分成 trees 份，然后再根据每条路上的最小值进行调整
		//对于某棵树上没有找到路，那么其最小值就是0，这样以来，就是在其他树的路径上去平分了
	//check if transaction works：
	//如果这棵树上有路，检查更新后的边权重是否满足原来的Creditlinks的最大最小要求，进行更新creditlinks的wt[1]
	//如果所有的路径上都成功，将更新后，那些容量为0的边记录一下
		CreditNetwork.zeroEdges.add(new Edge(src,dst));
	
	//compute metrics
	int[] res = new int[6+this.roots.length];//前6个是固定的要统计的值，后roots.length个值分别是各个树上的路
	res[0] = success?0:-1;
	res[1] = 统计总的路径长度;res[6+j] = paths[j].length-1;/-(paths[j].length-2);
	
	//receiver-landmarks
	//the sender and receiver both send only one message to each landmark forwarded by all nodes on the shortest path to the landmark.
	int d = sp.getDepth(dest);
	res[2] = res[2]+d*paths.length; //统计paths.length个值分别是各个树上的路；
		
	//landmarks-sender
	int d = sp.getDepth(src);
	res[3] = res[3]+d;
	
	//overall message count
	res[4] = res[1]+res[2]+res[3];
	res[4] = res[4]+2*(paths[j].length-1);//累加
	
	//delay 与src，dest 的depth 有关，具体怎么算的实在是看不明白
```





