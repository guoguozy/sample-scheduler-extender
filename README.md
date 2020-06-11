# scheduler-extender
kubernetes自定义scheduler-extender

## 自定义extender工作逻辑

### filter
filter函数调用podFitsOnNode函数，来判断当前pod是否适合放在这个节点上。判断依据为，一个随机数对6求余数，这个余数是否等于2。如果等于2，则证明这个pod可以放在该节点上。
```go
func podFitsOnNode(pod *v1.Pod, node v1.Node) (bool, []string, error) {
	var failReasons []string
	judge := (rand.Intn(100)%6 == 2)
  
	if judge {
		log.Printf("pod %v/%v fits on node %v\n", pod.Name, pod.Namespace, node.Name)
		return true, nil, nil
	}
	log.Printf("pod %v/%v does not fit on node %v\n", pod.Name, pod.Namespace, node.Name)

	failures := "It's not fits on this node."
	failReasons = append(failReasons, failures)
  
	return false, failReasons, nil
}
```

### prioritize
在prioritize函数中，每个节点按照在队列中的顺序打分，分数依次升高。
```go
//打分，排序
func prioritize(args schedulerapi.ExtenderArgs) *schedulerapi.HostPriorityList {
	pod := args.Pod
	nodes := args.Nodes.Items
	hostPriorityList := make(schedulerapi.HostPriorityList, len(nodes))
	
  	for i, node := range nodes {
		score := i+1
		log.Printf(PrioMsg, pod.Name, pod.Namespace, score)
		
    		hostPriorityList[i] = schedulerapi.HostPriority{
			Host:  node.Name,
			Score: score,
		}
	}
	return &hostPriorityList
}
```
