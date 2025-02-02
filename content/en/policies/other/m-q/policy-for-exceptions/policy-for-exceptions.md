---
title: "Policy for PolicyExceptions"
category: Sample
version: 1.9.0
subject: PolicyException
policyType: "validate"
description: >
    A PolicyException grants the applicable resource(s) or subject(s) the ability to bypass an existing Kyverno policy. Care should be taken to ensure that the allowed PolicyExceptions are scoped fine enough and according to your organization's operation. This is a Kyverno policy intended to provide guardrails for Kyverno PolicyExceptions and contains a number of rules which may help with these scoping best practices. These rules may be changed/removed depending on the exception practices to be implemented.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/m-q/policy-for-exceptions/policy-for-exceptions.yaml" target="-blank">/other/m-q/policy-for-exceptions/policy-for-exceptions.yaml</a>

```yaml
apiVersion: kyverno.io/v2beta1
kind: ClusterPolicy
metadata:
  name: policy-for-exceptions
  annotations:
    policies.kyverno.io/title: Policy for PolicyExceptions
    policies.kyverno.io/category: Sample
    policies.kyverno.io/minversion: 1.9.0
    kyverno.io/kyverno-version: 1.9.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/subject: PolicyException
    policies.kyverno.io/description: >-
      A PolicyException grants the applicable resource(s) or subject(s) the ability
      to bypass an existing Kyverno policy. Care should be taken to ensure that
      the allowed PolicyExceptions are scoped fine enough and according to your
      organization's operation. This is a Kyverno policy intended to provide guardrails
      for Kyverno PolicyExceptions and contains a number of rules which may help
      with these scoping best practices. These rules may be changed/removed depending
      on the exception practices to be implemented.
spec:
  validationFailureAction: Audit
  background: false
  rules:
  - name: single-policy
    match:
      any:
      - resources:
          kinds:
          - PolicyException
    validate:
      message: >-
        An exception is only allowed for a single policy.
        Please create a separate exception per policy.
      deny:
        conditions:
          any:
          - key: "{{request.object.spec.exceptions[] | length(@)}}"
            operator: GreaterThan
            value: 1
  - name: require-match-name
    match:
      any:
      - resources:
          kinds:
          - PolicyException
    validate:
      message: >-
        An exception must explicitly specify a name for a resource match.
      pattern:
        spec:
          match:
            =(any):
            - resources:
                names: "?*"
            =(all):
            - resources:
                names: "?*"
  - name: require-subject
    match:
      any:
      - resources:
          kinds:
          - PolicyException
    validate:
      message: >-
        An exception must explicitly specify a subject which is expected to create the resource.
      pattern:
        spec:
          match:
            =(any):
            - subjects: 
              - name: "?*"
            =(all):
            - subjects: 
              - name: "?*"
  - name: no-cross-namespace-exceptions
    match:
      any:
      - resources:
          kinds:
          - PolicyException
    validate:
      message: >-
        An exception can only be created in the same Namespace as the resource being excluded.
      deny:
        conditions:
          any:
          - key: "{{ request.object.spec.match.[any, all][].resources[].namespaces[] || `[]`}}"
            operator: AnyNotIn
            value: "{{ request.namespace }}"
  - name: namespaced-exceptions-only
    match:
      any:
      - resources:
          kinds:
          - PolicyException
    validate:
      message: >-
        An exception can only be created for a Namespaced resource, and a Namespace is required.
      foreach:
      - list: request.object.spec.match.[any, all][]
        deny:
          conditions:
            any:
            - key: "{{ element.resources.namespaces[] || `[]` | length(@) }}"
              operator: Equals
              value: 0
  - name: policy-namespace-match-polex-namespace
    match:
      any:
      - resources:
          kinds:
          - PolicyException
    validate:
      message: >-
        An exception may not be provided for a Namespaced Policy in another Namespace.
      foreach:
      - list: request.object.spec.exceptions[]
        preconditions:
          any:
          - key: "{{element.policyName}}"
            operator: Equals
            value: "*/*"
        deny:
          conditions:
            any:
            - key: "{{ element.policyName}}"
              operator: NotEquals
              value: "{{request.namespace}}/*"

```
