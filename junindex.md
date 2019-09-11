---
layout: default
---
 
---
# The Ugly Of Kubernetes For Loving Kubernetes More
---

## Prolog

Well, this talk is, rather than for criticizing Kubernetes (K8s in short), more of making interesting notes about the nasty and ugly parts of K8s so that hopefully it makes you able to utilize K8s in a proper way and helps you not to waste too much time in your own debugging. 

Although I assume you are familiar with at least the fundamentals of K8s, as a baseline of the talk, let me quickly review some relevant concepts and objectives in K8s in a few paragraphs. 

## Why K8s?

K8s is a platform that allows you to deploy container-based applications (a.k.a., micro-services) mostly (!) in a declarative way so that you can manage (i.e., deploy, watch, scale, upgrade, and delete) the life cycle of your apps at scale in a systematic way. K8s is designed mainly with the mindset that you just need to focus on your apps, not too much on how to manage your apps. At least so is it in theory (^^). With that said, K8s basically allows you to deal with abstracted layers, so called API objects, which can be controlled by K8s native API on top of your apps via "YAML files." 

Examples of API objects are Pod, Service, Deployment, ReplicaSet, etc. Pod is a basic unit of runtime where you can run your apps (or containers) at scale. Service is a way to expose your Pod to others. Deployment is to provide a declarative way to deploy your apps via other API objects (e.g., Pod, ReplicaSet, etc.). ReplicaSet is an API object which is controlled by Deployment to automatically create and watch replicas (i.e., copies of Pod), and recreate them upon failures of replicas. So a typical way to run your apps on K8s is to create a deployment that defines your Pod, and/or its Service to expose your Pod, with ReplicaSet. That's pretty much it!

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 3
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
```

Figure1. A Deployment to deploy an Nginx app with 3 replicas and a Service to expose them.

All right, I think it's enough for basic scaffolding. Now, it looks like everything would work well because the goal of K8s is crystal clear and API objects are well defined by understanding actual end-to-end workflows in managing apps. Well, not really in real life. I would say there would be many edge cases you may run into where you might not be certain as to which way would be a best practice. This argument could be applicable to K8s developers as well, I guess. What I mean by it is K8s developers would not be certain for such edge cases on which way could be a best practice to implement. 

## What Brought Up The Ugly In K8s?  

Earlier, I mentioned "YAML files" that are used for describing API objects in K8s. YAML files in K8s are an important method to provide K8s' Domain Specific Language (DSL). With YAML format, K8s has pre-defined structures (expandable though) for all K8s native API objects and its own interpretation on each entry or item in YAML files. The "interpretation" means a pre-defined and expected behavior behind scene on every single entry, executed by K8s. Now, we've finally arrived at a critical point that can possibly make K8s' DSL look ugly if such an interpretation in K8s is not well-designed. And unfortunately I am arguing that yes, it's horrible in K8s. Maybe we can ascribe all the ugly to the inherent limitations of YAML format itself. However, in this talk, let's focus on what the ugly are rather than why we got them.

### Declarative vs. Imperative

Some of you may have noticed that I said "K8s is a platform …. **mostly** (!) in a declarative way." Yes, it targets at providing a declarative framework for most of the cases. However, in reality, such looking-declarative cases involve some degree of imperative situations.

