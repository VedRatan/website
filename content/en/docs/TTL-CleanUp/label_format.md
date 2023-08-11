---
title: Label Format
description: Supported format for the label.
weight: 30
---

The ttl controller is purposefully engineered to exclusively manage resources equipped with a specific label: **cleanup.kyverno.io/ttl**. This label serves to establish a time-to-live duration for a resource, effectively functioning as a countdown timer from the resource's creation time. The value assigned to this label is represented as a string and can be expressed in minutes, hours, or days. For instance, examples of valid labels include: `cleanup.kyverno.io/ttl: 30m` and `cleanup.kyverno.io/ttl: 30d`. Notably, this label system also accommodates the provision of an absolute date and time for when the cleanup process should occur. These absolute timestamps adhere to the ISO 8601 standards, as demonstrated by labels like `cleanup.kyverno.io/ttl: 2022-08-04T00:30:00Z` and `cleanup.kyverno.io/ttl: 2022-09-30`.

Furthermore, it's important to mention that the ttl controller incorporates a safeguard against invalid label values. If a user attempts to set a label value that does not conform to the specified format, a warning mechanism will be triggered. This warning serves as an alert to the user, indicating that the provided label does not adhere to the supported format. This validation process is carried out by the **validation-webhook**, a component configured to monitor resources bearing the **cleanup.kyverno.io/ttl** label. The webhook diligently inspects the label values to ensure they align with the prescribed format. If any non-conforming values are detected, a warning is raised to maintain the integrity of the labeling system.