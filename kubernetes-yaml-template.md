# Template yaml pada kubernetes

1. Deploy Database
   * Membuat Persistent Volume

      ```yaml
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
       name: cloud-ssd
      provisioner: kubernetes.io/aws-ebs
      parameters:
       type: gp2
      ```
   * Membuat Persisten Volume Claim

      ```yaml
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
       name: mysql-pvc
      spec:
       storageClassName: cloud-ssd
       accessModes:
        - ReadWriteOnce
       resources:
        requests:
         storage: 7Gi
      ```
   * Deploy MySQL Pod

      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
       name: mysql-dc
      spec:
       selector:
        matchLabels:
         app: mysql-dc
       replicas: 1
       template: 
        metadata:
         labels:
          app: mysql-dc
        spec:
         containers:
         - name: mysql-dc
           image: mysql:latest
           env:
            - name: MYSQL_ROOT_PASSWORD
              value: root
            - name: MYSQL_USER
              value: super
            - name: MYSQL_PASSWORD
              value: super
            - name: MYSQL_DATABASE
              value: test2
           volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /data/db
         volumes:
         - name: mysql-persistent-storage
           persistentVolumeClaim:
            claimName: mysql-pvc
      ```
   * Create MySQL Service

      ```yaml
      kind: Service
      apiVersion: v1
      metadata:
       name: mysql-svc
      spec:
       selector:
        app: mysql-dc
       ports:
        - name: mysqlport
          port: 3306
       type: ClusterIP
      ```

   * Create Application Pod
      ```yaml
      # Pod
      apiVersion: apps/v1
      kind: Deployment
      metadata:
       name: api-dc
      spec:
       selector:
        matchLabels:
         app: api-dc
       replicas: 1
       template:
        metadata:
         labels:
          app: api-dc
        spec:
         containers:
         - name: api-dc
           image: ridlwan/simple-spring-api:2.0
           env:
           - name: SPRING_DATASOURCE_URL
             value: jdbc:mysql://mysql-svc.default.svc.cluster.local/test2?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
           - name: SPRING_DATASOURCE_USERNAME
             value: super
           - name: SPRING_DATASOURCE_PASSWORD
             value: super
           ports:
           - containerPort: 8000
             name: api-dc
      ```
      
     Dijelaskan bahwa endpont datasource adalah `mysql-svc.default.svc.cluster.local` dimana:  
     `svc.cluster.local` didapatkan dari nama default lokal cluster pada kubernetes.  
     `default` didapatkan dari nama `namespace`.  
     `mysql-svc` didapatkan dari nama `service`.

   * Create Application Service
      ```yaml
      # Service
      apiVersion: v1
      kind: Service
      metadata:
       name: spring-api-svc
      spec:
       selector:
        app: api-dc
       ports:
        - name: http
          port: 80
          targetPort: 8000
          protocol: TCP
          name: http
       type: LoadBalancer
      ```
2. Setting Role
   * Create Role
      ```yaml
      # Role
      kind: Role
      apiVersion: rbac.authorization.k8s.io/v1
      metadata:
       name: new-joiner
       namespace: default
      rules:
      - apiGroups: ["", "apps"] # Core API and apps
        resources: ["*"] # pods, services, deployments
        verbs: ["get","list","watch"]
      ```  
   * Create RoleBinding 
      ```yaml
      # RoleBinding
      kind: RoleBinding
      apiVersion: rbac.authorization.k8s.io/v1
      metadata:
       name: put-specific-user-or-users-into-new-joiner-role
       namespace: default
      subjects:
      - kind: User
        name: francis-linux-login-name
      roleRef:
       kind: Role
       name: new-joiner
       apiGroup: rbac.authorization.k8s.io
      ```  
   * Create User
      ```bash
      [ec2-user@ip-x-x-x-x ~]$ sudo useradd francis-linux-login-name
      [ec2-user@ip-x-x-x-x ~]$ sudo passwd francis-linux-login-name
      [ec2-user@ip-x-x-x-x ~]$ su - francis-linux-login-name
      ``` 
   * Edit Kubernetes Configuration
   * Show The Kubernetes API by using
      ```bash
      [ec2-user@ip-x-x-x-x ~]$ kubectl config view
      ``` 
   * Copy `clusters.cluster.server` value
      ```bash
        server: https://api-xxx-xx-xx-xxx-xxx.ap-southeast-1.elb.amazonaws.com
      ``` 

   * Edit Kubernetes inside of `francis-linux-login-name` user
      ```bash
      [ec2-user@ip-x-x-x-x ~]$ su - francis-linux-login-name
      [francis-linux-login-name@ip-x-x-x-x ~]$ kubectl config set-cluster xxx.xxx.xxx --server=https://api-xxx-xx-xx-xxx-xxx.ap-southeast-1.elb.amazonaws.com
      [francis-linux-login-name@ip-x-x-x-x ~]$ kubectl config set-context mycontext --user francis-linux-login-name --cluster xxxx.xxx.xxx
      [francis-linux-login-name@ip-x-x-x-x ~]$ kubectl config use-context mycontext
      ``` 
   * Setting X509 Certificate for `francis-linux-login-name` user
      ```bash
      [ec2-user@ip-x-x-x-x ~]$ openssl -out private-key-francis 2048
      [ec2-user@ip-x-x-x-x ~]$ openssl req -new -key private-key-francis.key -out req.csr -subj "/CN=francis-linux-login-name/O=francis-linux-login-name"
      [ec2-user@ip-x-x-x-x ~]$ aws s3 cp s3://xxxx-xxxx-xxxx/xxx.xxx.xxx/pki/private/ca/213132232132131231312.key kubernetes.key
      [ec2-user@ip-x-x-x-x ~]$ aws s3 cp s3://xxxx-xxxx-xxxx/xxx.xxx.xxx/pki/issued/ca/1212121212121212121212.crt kubernetes.crt
      [ec2-user@ip-x-x-x-x ~]$ openssl x509 -req -in req.csr -CA kubernetes.crt -CAkey kubernetes.key -CAcreateserial -out francis.crt -days 365
      [ec2-user@ip-x-x-x-x ~]$ sudo mkdir /home/francis-linux-login-name/.certs
      [ec2-user@ip-x-x-x-x ~]$ sudo cp kubernetes.crt /home/francis-linux-login-name/.certs
      [ec2-user@ip-x-x-x-x ~]$ sudo cp francis.crt /home/francis-linux-login-name/.certs
      [ec2-user@ip-x-x-x-x ~]$ sudo cp private-key-francis.crt /home/francis-linux-login-name/.certs
      [ec2-user@ip-x-x-x-x ~]$ sudo chown -R francis-linux-login-name:francis-linux-login-name /home/francis-linux-login-name/.certs/
      [ec2-user@ip-x-x-x-x ~]$ su - francis-linux-login-name
      [francis-linux-login-name@ip-x-x-x-x ~]$ cd .certs
      [francis-linux-login-name@ip-x-x-x-x ~]$ kubectl config set-credentials francis-linux-login-name --client-certificate=francis.crt --client-key=private-key-francis.key
      [francis-linux-login-name@ip-x-x-x-x ~]$ kubectl config set-cluster xxxx.xxx.xxxx --certificate-authority=kubernetes.crt
      [francis-linux-login-name@ip-x-x-x-x ~]$ kubectl config view
      [francis-linux-login-name@ip-x-x-x-x ~]$
      [francis-linux-login-name@ip-x-x-x-x ~]$
      ``` 
3. Under Construction
