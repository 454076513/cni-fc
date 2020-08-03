# A flow control  CNI plugin

### Linux Environmental dependence：
tc、iptables and jq

### Plug-in storage location：
/opt/cni/bin

### Kubernetes Config file /etc/cni/net.d：
```json
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
        "name": "Flow control",
        "type": "fc",
        "rules":{
            "drop":[
                {
                    "ip":"172.17.0.2",
                    "port":"80"
                },
                {
                    "ip":"172.17.0.9",
                    "port":"80"
                }
            ],
            "reject":[
                {
                    "ip":"172.17.0.2/24",
                    "port":"80"
                }
            ],
            "accept":[
                {
                    "ip":"172.17.0.2/24",
                    "port":"80"
                }
            ],
            "bandwidth":[
                {
                    "ip":"172.17.0.2",
                    "port":"80",
                    "rate":"1kbit",
                    "ceil":"5kbit",
                    "burst":"10kbit",
                    "prio":"16"

                },
                {
                    "ip":"172.17.0.3",
                    "port":"80",
                    "rate":"1kbit",
                    "ceil":"5kbit",
                    "burst":"10kbit",
                    "prio":"16"

                }
            ]
        }
    }
  ]
}
```

