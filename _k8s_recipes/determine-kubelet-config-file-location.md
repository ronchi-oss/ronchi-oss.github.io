---
layout: page
title: Determining kubelet config file location
permalink: /kubernetes/kubelet-config-file-location/
parent: ["/kubernetes/", "Kubernetes"]
---
```
sudo systemctl status kubelet
```

That path is part of the output of the `CGroup` part:

```
/usr/bin/kubelet [...] --config=/var/lib/kubelet/config/config.yaml
```

