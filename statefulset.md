**what is the stateful application in expense project?**
- MySQL is the stateful application

**statefulset vs deployment**
- Statefulset is for DB related application
- Deployment is for stateless application
- Statefulset will have headless service along with normal service. It requires pv and pvc objects
- Deployment will not have headless service
- Statefulset pods will be created in orderly manner
- Statefulset will keep its pod identity. Pod names will be created as -0, -1, -2 etc.,
- In statefulset, after creating and running one replica then only another replica will be created.  

![alt text](images/headless_service.drawio.svg)

- Here, MySQL master has different nodes and every node has their own individual database
- If one node received any data replication (like creation, updation, deletion etc) it informs other nodes to do the same
- For example, node-1 should know which nodes are there in the cluster and there IP address.

**what is headless service?**
- Headless service will not have cluster IP, if anyone does nslookup on headless service it will give all endpoints 

Here we are rewriting the expense-project 
1. create expense namespace (Admin Activity)
2. install ebs drivers 
3. create ebs sc 
4. give eks nodes ebs permissions
5. create pvc and create statefulset 

**Namespace**
```
apiVersion: v1
kind: Namespace
metadata:
  name: expense
  labels:
    project: expense
    environment: dev
```
```
kubectl apply -f 01-namespace.yml
```

**Driver Installation**
- Go to the github location and copy the following command and install drivers
```
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.36"
```

**Create EBS Storage Class**
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expense-ebs
reclaimPolicy: Retain
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer 
```

**Give eks nodes ebs permissions**
- Go to IAM Roles in node and select `AmazonEBSCSIDriverPolicy`

**Create PVC and Statefulset**
- Create headless service for MySQL
```
kind: Service
apiVersion: v1
metadata:
  name: mysql-headless
  namespace: expense
spec:
  clusterIP: None # for headless service there is no cluster IP
  selector:
    project: expense
    component: mysql
    tier: db
  ports:
  - protocol: TCP
    port: 3306 # service port
    targetPort: 3306
```

- Along with headless service, we also have to create normal service

```
kind: Service
apiVersion: v1
metadata:
  name: mysql
  namespace: expense
spec:
  selector:
    project: expense
    component: mysql
    tier: db
  ports:
  - protocol: TCP
    port: 3306 # service port
    targetPort: 3306
```

- Now, we will create statefulset

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: expense
spec:
  selector:
    matchLabels:
      project: expense
      component: mysql
      tier: db
  serviceName: "mysql-headless" # this is the headless service should be mentioned for statefulset
  replicas: 2 # 
  template:
    metadata:
      labels:
        project: expense
        component: mysql
        tier: db
    spec:
      containers:
      - name: mysql
        image: joindevops/mysql:v1
        volumeMounts:
        - name: mysql
          mountPath: /var/lib/mysql
  # This is PVC defnition, directly mentioned here
  volumeClaimTemplates:
  - metadata:
      name: mysql
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "expense-ebs"
      resources:
        requests:
          storage: 1Gi
```
- In statefulset, it is mandatory to give PVC
- Here, storageclassname should be same as provided in ebs-sc.yml
- Here, volumemounts name and volume claim templates name should be same
- Location of mysql (mountpath) is /var/lib/mysql
- In whole statefulset, the difference is statefulset will have headless service which is explicitly attached and it will have PVC for sure

```
cd mysql/
```
```
kubectl apply -f manifest.yaml
```
```
kubens expense
```
- This is set expense as namespace
```
kubectl get pods
```
```
kubectl get pv,pvc
```
```
kubectl get svc
```
- Now, go to any one of the pod
```
kubectl exec -it mysql-0 -- bash
```
- Imagine pod got a request of create operation and it should send to remaining pods. Before this it should know which pods are there in the cluster.
```
cat /etc/*release
```
- We use this command to know the name of the OS to install nslookup
- If it is oracle linux server, then use the following commands
```
microdnf update -y
```
```
microdnf install -y bind-utils
```
```
nslookup mysql-headless
```
- It will display the POD IP addresses
```
kubectl get pods -o wide
```
- Here, we can see the PODS and its IP Addresses
- If we do nslookup in normal service, we have got cluster IP address.
- Internally, if pod gets create operation, then nslookup headless service run and see the other pods in the cluster and will get connected to them.

Q. Why should we use headless service and what is headless service?

A. Headless service will not have any cluster IP attached to it. In database related application if any create, updated and delete operations comes to one node it has to inform for all other nodes in the cluster. First, it will do nslookup of headless service, so that it can find all the IP addressess of other nodes and then inform to them.

- In deployment, the names of the pods are randomly created. But in stateful, it will create in orderly manner.

- Now, do backend and frontend 
```
- cd ../backend/
```
```
kubectl apply -f manifest.yml
```
```
- cd ../frontend/
```
```
kubectl apply -f manifest.yml
```
```
kubectl get pods
```
```
kubectl get svc
```
- Copy frontend Loadbalancer IP address 
- We need to open all ports (All TCP) in cluster secuirty group because we don't know which port it will open but will we reestrict in future.

- If we want to check whether the data replictaion is successfully done or not, then go to all the nodes and check.

```
kubectl exec -it mysql-1 -- bash
```
```
mysql -u root -pExpense@1
```
```
use transcations;
```
```
select * from transactions;
```

- Now, check for second node

```
kubectl exec -it mysql-0 -- bash
```
```
mysql -u root -pExpense@1
```
```
use transcations;
```
```
select * from transactions;
```












