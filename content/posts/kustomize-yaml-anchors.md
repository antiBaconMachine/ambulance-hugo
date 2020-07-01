+++
author = "Ollie Edwards"
categories = ["kubernetes", "yaml", "kustomize"]
date = "2020-07-01T16:40:43+01:00"
draft = false
title = "Kustomize, Yaml Anchors and You"
slug = "kustomize-yaml-anchors"
+++

Kustomize is a tool for composing kubernetes manifests. It helps you to not repeat yourself when configuring many resources which are similar and staging those resources accross environments whciih are similar.

It is not a templating language.

In general this is a good thing. There are other tools whcih can help you with templating, kustomize just helps with boilerplate reduction.

I ran into a situation recently where I found it desirable to have both a CronJob and a Job resource which were mighty similar. A CronJob resource in kubernetes might look like

```language-yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  concurrencyPolicy: Forbid
  jobTemplate:
    metadata:
      name: my-job
    spec:
      template:
        spec:
          containers:
            image: busybox
            name: job
            foo: bar
          initContainers:
            image: busybox
            name: jobinit
          restartPolicy: Never
  schedule: '*/1 * * * *'
  startingDeadlineSeconds: 300
```

And a job might look something like

```language="yaml"
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    metadata:
      name: my-job
    spec:
      template:
        spec:
          containers:
            image: busybox
            name: job
            foo: bar
          initContainers:
            image: busybox
            name: jobinit
          restartPolicy: Never
```

So, extremely similar, a lot of shared boilerplate. Kustomize generally works on the principle of applying merge patches to existing resources. It doesn't direcly support any kind of resource sharing/loading that would help us to reuse this boilerplate. yaml does have something that will help us though, anchors and aliases. We can use these together with the core kubernetes resouce `List` to share the job template between the two resources, like so:

```language-yml
apiVersion: v1
kind: List
items:
- apiVersion: batch/v1beta1
  kind: CronJob
  metadata:
    name: my-cronjob
  spec:
    concurrencyPolicy: Forbid
    jobTemplate:
      metadata:
        name: my-job
      spec: &jobspec
        template:
          spec:
            containers:
              image: busybox
              name: job
              foo: bar
            initContainers:
              image: busybox
              name: jobinit
            restartPolicy: Never
    schedule: '*/1 * * * *'
    startingDeadlineSeconds: 300
- apiVersion: batch/v1
  kind: Job
  metadata:
    name: data-vaults-templated-job
  spec: *jobspec
```

Note the anchor `&jobspec` and alias `*jobspec`. Using this as a base for a kustomization will output a compound yaml stream containig both resources with the job alias fully inflated.

What if you only want to use one of these resources at a time?

For this the best solution I've found is to specify an overlay kustomization which deletes the resource you are not interested in. You can achieve this using either supported patch format, I'll use a strategic merge patch for this example. The overlay kustomization.yaml looks like this:

```language=yml
bases:
  - ../base
patchesStrategicMerge:
  - deleteCronJob.yml
```

The deletion patch looks like this

```language="yml"
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cronjob
$patch: delete
```

When we run this through `kustomize build` the output is a single Job resource.

Kustomize is a great tool, which can suffer from quite poor documentation, I pieced this together from various PRs, issues and source. It is also deliberately very limited, but if you stick with it, I think you can make some really nice terse compositions which certainly reduce boilerplate and increase readibility.
