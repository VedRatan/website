---
title: "Replace Image Registry"
category: Sample
version: 1.6.0
subject: Pod
policyType: "mutate"
description: >
    Rather than blocking Pods which come from outside registries, it is also possible to mutate them so the pulls are directed to approved registries. In some cases, those registries may function as pull-through proxies and can fetch the image if not cached. This policy mutates all images either in the form 'image:tag' or 'registry.corp.com/image:tag' to be `myregistry.corp.com/`. Any path in the image name will be preserved. Note that this mutates Pods directly and not their controllers. It can be changed if desired but if so may need to not match on Pods.      
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/rec-req/replace-image-registry/replace-image-registry.yaml" target="-blank">/other/rec-req/replace-image-registry/replace-image-registry.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: replace-image-registry
  annotations:
    policies.kyverno.io/title: Replace Image Registry
    pod-policies.kyverno.io/autogen-controllers: none
    policies.kyverno.io/category: Sample
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.7.2
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      Rather than blocking Pods which come from outside registries,
      it is also possible to mutate them so the pulls are directed to
      approved registries. In some cases, those registries may function as
      pull-through proxies and can fetch the image if not cached.
      This policy mutates all images either in the form 'image:tag' or
      'registry.corp.com/image:tag' to be `myregistry.corp.com/`. Any
      path in the image name will be preserved. Note that this mutates Pods
      directly and not their controllers. It can be changed if desired but
      if so may need to not match on Pods.      
spec:
  background: false
  rules:
    - name: replace-image-registry-pod-containers
      match:
        any:
        - resources:
            kinds:
            - Pod
      mutate:
        foreach:
        - list: "request.object.spec.containers"
          patchStrategicMerge:
            spec:
              containers:
              - name: "{{ element.name }}"
                image: "{{ regex_replace_all('^(localhost/|(?:[a-z0-9]+\\.)+[a-z0-9]+/)?(.*)$', '{{element.image}}', 'myregistry.corp.com/$2' )}}"
    - name: replace-image-registry-pod-initcontainers
      match:
        any:
        - resources:
            kinds:
            - Pod
      preconditions:
        all:
        - key: "{{ request.object.spec.initContainers[] || `[]` | length(@) }}"
          operator: GreaterThanOrEquals
          value: 1
      mutate:
        foreach:
        - list: "request.object.spec.initContainers"
          patchStrategicMerge:
            spec:
              initContainers:
              - name: "{{ element.name }}"
                image: "{{ regex_replace_all('^(localhost/|(?:[a-z0-9]+\\.)+[a-z0-9]+/)?(.*)$', '{{element.image}}', 'myregistry.corp.com/$2' )}}"

```
