---
title: "Limit Containers per Pod"
category: Sample
version: 1.3.6
subject: Pod
policyType: "validate"
description: >
    Pods can have between one and a number of different containers included which are tightly coupled. It may be desired to limit the amount of containers that can be in a single Pod to control best practice application or so policy can be applied consistently. This sample checks all Pods to ensure they have no more than four containers.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/limit_containers_per_pod.yaml" target="-blank">/other/limit_containers_per_pod.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: limit-containers-per-pod
  annotations:
    policies.kyverno.io/title: Limit Containers per Pod
    policies.kyverno.io/category: Sample
    policies.kyverno.io/minversion: 1.3.6
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Pods can have between one and a number of different containers included which
      are tightly coupled. It may be desired to limit the amount of containers that
      can be in a single Pod to control best practice application or so policy can
      be applied consistently. This sample checks all Pods to ensure they have
      no more than four containers.
spec:
  background: false
  validationFailureAction: enforce
  rules:
  - name: limit-containers-per-pod-controllers
    match:
      resources:
        kinds:
        - Deployment
        - DaemonSet
        - Job
        - StatefulSet
    preconditions:
      all:
      - key: "{{request.operation}}"
        operator: Equal
        value: CREATE
    validate:
      message: "Pods can only have a maximum of 4 containers."
      deny:
        conditions:
          any:
          - key: "{{request.object.spec.template.spec.containers[] | length(@)}}"
            operator: GreaterThan
            value: "4"
  - name: limit-containers-per-pod-bare
    match:
      resources:
        kinds:
        - Pod
    preconditions:
      all:
      - key: "{{request.operation}}"
        operator: Equal
        value: CREATE
    validate:
      message: "Pods can only have a maximum of 4 containers."
      deny:
        conditions:
          any:
          - key: "{{request.object.spec.containers[] | length(@)}}"
            operator: GreaterThan
            value: "4"
  - name: limit-containers-per-pod-cronjob
    match:
      resources:
        kinds:
        - CronJob
    preconditions:
      all:
      - key: "{{request.operation}}"
        operator: Equal
        value: CREATE
    validate:
      message: "Pods can only have a maximum of 4 containers."
      deny:
        conditions:
          any:
          - key: "{{request.object.spec.jobTemplate.spec.template.spec.containers[] | length(@)}}"
            operator: GreaterThan
            value: "4"

```