# k8s的网络策略（Network Policy）

Pod之间通信需要网络，为了安全考虑，需要限制Pod之间的通信，k8s提供了NetworkPolicy资源来实现网络策略。
NetworkPolicy工作在3层（网络）和4层（传输）。

由于Pod是可以随时创建和销毁的。IP随时可能改变，因此NetworkPolicy不能依赖IP地址来控制Pod之间的通信。转而使用标签选择器 (Label Selectors) 来动态锁定目标资源并应用网络规则。

## Network Policy所做的事情

Network Policy 资源定义了哪些Pod可以互相通信，以及如何通信。

## Network Policy的策略

## Network Polic的运行机制

## Network Policy的YAML
