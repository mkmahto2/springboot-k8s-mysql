## bulid the image from Dockerfile 
~~~
docker run -it springboot-k8s-mysql:1.0 .
~~~
## create configmap file
mysql-configmap.yml
~~~

apiVersion: v1
kind: ConfigMap
metadata:
  name: db-conf
data:
 host: mysql
 name: test  

~~~

create secret file 
mysqldb-credentials.yml
~~~


apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
data:
 username: dGVzdHVzZXI=
 password: dGVzdHVzZXJAMTIz

~~~
mysqldb-root-credentials.yml

~~~

apiVersion: v1
kind: Secret
metadata:
  name: db-root-credentials
data:
 password: YWRtaW4xMjM=

~~~


Create deployment.yml file

~~~
 
kind: Service
apiVersion: v1
metadata:
  name: springboot-k8s-mysql
  labels:
    name: springboot-k8s-mysql
spec:
  ports:
    - nodePort: 30163 
      port: 8080      
      targetPort: 8080  
      protocol: TCP
  selector:           
    app: springboot-k8s-mysql
  type: NodePort       

---
apiVersion: apps/v1 
kind: Deployment    
metadata:              
  name: springboot-k8s-mysql
spec:                
  selector:         
    matchLabels:
      app: springboot-k8s-mysql
  replicas: 3        
  template:
    metadata:
      labels:        
        app: springboot-k8s-mysql
    spec:
      containers:
        - name: springboot-k8s-mysql
          image: springboot-k8s-mysql:1.0
          ports:
            - containerPort: 8080                
          env:   # Setting Enviornmental Variables
          - name: DB_HOST   # Setting Database host address from configMap
            valueFrom: 
              configMapKeyRef:
                name: db-conf  # name of configMap
                key: host
          - name: DB_NAME  # Setting Database name from configMap
            valueFrom:
              configMapKeyRef:
                name: db-conf 
                key: name
          - name: DB_USERNAME  # Setting Database username from Secret
            valueFrom:
              secretKeyRef:
                name: db-credentials # Secret Name
                key: username
          - name: DB_PASSWORD # Setting Database password from Secret
            valueFrom:
              secretKeyRef:
                name: db-credentials
                key: password

~~~

application.yml
~~~
spring:
    datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://${DB_HOST}/${DB_NAME}?useSSL=false
        username: ${DB_USERNAME}
        password: ${DB_PASSWORD}
        hikari:
            initialization-fail-timeout: 0        
    jpa:
        database-platform: org.hibernate.dialect.MySQL5Dialect
        generate-ddl: true
        show-sql: true
        hibernate:
            ddl-auto: create

~~~
