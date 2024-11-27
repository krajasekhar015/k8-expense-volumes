**what is the stateful application in expense project?**
- MySQL is the stateful application

**statefulset vs deployment**
- Statefulset is for DB related application
- Deployment is for stateless application
- Statefulset will have headless service along with normal service. It requires pv and pvc objects
- Deployment will not have headless service 

![alt text](images/headless_service.drawio.svg)

- Here, MySQL master has different nodes and every node has individual their own database
- If one node received any data replication (like creation, updation, deletion etc) it informs other nodes to do the same

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






