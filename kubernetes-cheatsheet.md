
To create a pod:
kubectl create -f pods/monolith.yaml

To get the current pods:
=======================
kubectl get pods
kubectl get pods -o wide
kubectl get pods -l "app=monolith" --> To filter bassed on a label
kubectl get pods healthy-monolith -w
kubectl get pods secure-monolith --show-labels
To get pod-name:
================
kubectl get pods -l "app.kubernetes.io /component=jenkins-master" -o jsonpath="{.items[0].metadata.name}"
Matching multiple labels:
==========================
kubectl get pods -n production -l app=gceme -l role=f rontend


To get details on a particular pod:
===================================
kubectl describe pods <pod-name>


To map local port 10080 to connect to monolith pod's port 80:
============================================================
kubectl port-forward monolith 10080:80

To get logs on a pod:
====================
kubectl logs monolith
kubectl logs -c nginx secure-monolith --> To view logs of nginx container in secure-monolith pod

To run interactive shell inside a Pod:
======================================
kubectl exec monolith --stdin --tty -c monolith-container-name /bin/sh  

To create secrets:
==================
kubectl create secret generic tls-certs --from-file=tls/

To create configMap:
==================
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf

To create a service:
====================
kubectl create -f services/monolith.yaml


To add a label to a pod:
=======================
kubectl label pods secure-monolith 'secure=enabled'

To remove the above label:
==========================
kubectl label pods secure-monolith secure-


To create a deployment that abstracts out pods, u first create a delpoyment.
Then create a service.
============================================================================
kubectl create -f deployments/auth.yaml
kubectl create -f services/auth.yaml


To get the current replica sets:
================================
kubectl get replicasets

To scale up/down a deployment:
==============================
kubectl scale deployments hello --replicas=3


To apply config changes on an existing resource:
(Ex: Replicateset change; app image upgrade through version, etc)
=============================================================
kubectl apply -f deployments/frontend.yaml

To control rolling upgrade:
===========================
kubectl rollout pause deployment hello
kubectl rollout resume deployment hello
Use the kubectl rollout undo deployment hello


To get documentation:
=====================
kubectl explain pods
kubectl explain pods.spec.containers


TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
curl http://127.0.0.1:10080/secure



Pod liveless failure ==> Results in pod reboot
Pod readyness failure ==> Results in pod taken out of LB



- Deployment enables declarative updates for Pods and ReplicaSets.
  'create deployment' automatically creates a replicaset 

kubectl get replicasets

kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"

curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`


To do a rolling update:
=======================
Deployments update images to new versions through rolling updates. When a deployment is updated with a new version, it creates a new ReplicaSet and slowly increases the number of replicas in the new ReplicaSet as it decreases the replicas in the old ReplicaSet.

kubectl edit deployment hello

kubectl rollout history deployment/hello  <-- Gives history of rolling update for deployment named hello.

kubectl rollout pause deployment/hello
kubectl rollout status deployment/hello
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
kubectl rollout resume deployment/hello
kubectl rollout undo deployment/hello <-- TO GO back to previous version

Two types of Updates: Canary as incremental, and Blue-Green as Full-Flip.
=========================================================================

Canary Deployment:
 Deploy a new version of the app with just one pod with same label as the prev.
 With LB, the canary deployment with single pod only gets 1/N of the traffic.
 After validation, u can increase replicaset of canary deployment, and then retire the old deployment.  


Blue-Green Deployment:
Here, initially there is a blue-deployment with Version-1. The service is pointing to Version-1 through the label selector.
Then do a new deployment with newer version-2 called Green-Deployment. Change the service label to now use Version-2. Service will start using the pods in the newer deployment. Once satisfied, the older deployment can be retired.




To create a name-space:
kubectl create ns production

To extract IP on a service:
==========================
export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)

kubectl get namespace

kubectl run: creates a pod or deployment. if replicatset is provided, it creates a deployment.

kubectl expose: creates a service

Service Types 
 - ClusterIP:  The service gets an IP within the cluster, so cannot be accessed from outside the cluster. Hence a port-forward command on the local cluster node would direct the traffic to the clusterIP.

 - NodePort: Service is exposed to outside the cluster, but goes through the cluster Nodes. A static port is exposed on each node.
   Typically 3 ports are listed in the yaml file:
   (i) nodePort - a static port assigned to each node in the cluster
   (ii) port - port exposed internally in the cluster
   (iii) targetPort - the container port to send requests to. this is the
                      port containers are listening to.

 - LoadBalancer: Service is accessible externally through a load-balancer.
   Typically this is provided by cloud providers.



Probes:
-livenessProbe
-readinessProbe
-startupProbe


ResourceQuota works at the aggregate: i.e. sum of all 
resources within a namespace cannot exceed the quota limit.
ResourceQuota:  (can be applied at the Namespace).
    requests.cpu:
    requests.memory:
    limits.cpu:
    limits.memory:


LimitRange gets applied to each of the containers in a namespace.
You can control the min and max resource for each of the containers in the namespace.
LimitRange:
    limits:
    - defaults:
        cpu:
        memory:
      defaultRequest:
        cpu:
        memory:
      min:
        cpu:
        memory:
      max:
        cpu:
        memory:
      type: Container
      
     
    

