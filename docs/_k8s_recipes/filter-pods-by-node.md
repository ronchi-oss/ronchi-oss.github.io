---
layout: page
title: Filter pods by node
permalink: /kubernetes/filter-pods-by-node/
---

```sh
kubectl get pods -A --field-selector spec.nodeName=<node-name>
```

References:

* [https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/]()
