## GKE upgrade

* Upgrade GKE version could lead to all nodes (pool) unavailable up to 30m (tested with 2 nodes).
* If the GKE cluster has single zone, all nodes will be replaced when update regardless `max_unavailable` and `max_surge` setting.
