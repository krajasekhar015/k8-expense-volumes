**what is the stateful application in expense project?**
- MySQL is the stateful application

**statefulset vs deployment**
- Statefulset is for DB related application
- Deployment is for stateless application
- Statefulset will have headless service along with normal service. It requires pv and pvc objects
- Deployment will not have headless service 


- Here, MySQL master has different nodes and every node has individual their own database
- If one node received any data replication (like creation, updation, deletion etc) it informs other nodes to do the same

**what is headless service?**
- Headless service will not have cluster IP, if anyone does nslookup on headless service it will give all endpoints 

