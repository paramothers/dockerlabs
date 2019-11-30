# Kubernetes ReplicaSet

- A Kubernetes pod serves as a deployment unit for the cluster. 
- It may contain one or more containers. 
- However, containers (and accordingly, pods) are short-lived entities.
- A container hosting a PHP application, for example may experience an unhandled code exception causing the process to fail, effectively 
crashing the container. Of course, the perfect solution for such a case is to refactor the code to properly handle exceptions. 
- But, till that happens we need to keep the application running and the business going. In other words, we need to restart the pod 
whenever it fails. 
- In parallel, developers are monitoring, investigating and fixing any errors that make it crash. 
- At some point, a new version of the pod is deployed, monitored and maintained. It’s an ongoing process that is part of the DevOps practice.


Another requirement is to keep a predefined number of pods running. If more pods are up, the additional ones are terminated. 
Similarly, of one or more pods failed, new pods are activated until the desired count is reached.

A Kubernetes ReplicaSet resource was designed to address both of those requirements. 
It creates and maintains a specific number of similar pods (replicas). 

Under this lab, we’ll discuss how we can define a ReplicaSet and what are the different options that can be used for fine-tuning it.

# How Does ReplicaSet Manage Pods?

- In order for a ReplicaSet to work, it needs to know which pods it will manage so that it can restart the failing ones or kill the unneeded. 
- It also requires to understand how to create new pods from scratch in case it needs to spawn new ones.

A ReplicaSet uses labels to match the pods that it will manage. It also needs to check whether the target pod is already managed by another controller (like a Deployment or another ReplicaSet). So, for example if we need our ReplicaSet to manage all pods with the label role=webserver, the controller will search for any pod with that label. It will also examine the ownerReferences field of the pod’s metadata to determine whether or not this pod is already owned by another controller. If it isn’t, the ReplicaSet will start controlling it. Subsequently, the ownerReferences field of the target pods will be updated to reflect the new owner’s data.

To be able to create new pods if necessary, the ReplicaSet definition includes a template part containing the definition for new pods. 

# Creating Your First ReplicaSet

```
git clone https://github.com/collabnix/dockerlabs
cd dockerlabs/workshop/replicaset101
```

```
kubectl apply -f nginx_replicaset.yaml
```

```
kubectl get rs
```

```
NAME   DESIRED   CURRENT   READY   AGE
web	4     	4     	4   	2m
```

# A Peep into the ReplicaSet definition file

Let’s examine the definition file that was used to create our ReplicaSet:

- The apiVersion for this object is currently app/v1
- The kind of this object is ReplicaSet
- In the metadata part, we define the name by which we can refer to this ReplicaSet. We also define a number of labels through which we can identify it.
- The spec part is mandatory in the ReplicaSet object. It defines:
- The number of replicas this controller should maintain. It default to 1 if it was not specified.
- The selection criteria by which the ReplicaSet will choose its pods. Be careful not to use a label that is already in use by another controller. Otherwise, another ReplicaSet may acquire the pod(s) first. Also notice that the labels defined in the pod template (spec.template.metadata.label) cannot be different than those defined in the matchLabels part (spec.selector).
- The pod template is used to create (or recreate) new pods. It has its own metadata, and spec where the containers are specified. You can refer to our article for more information about pods.

