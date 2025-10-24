# In this repo, we are using statefulSets and creating RoboShop Databases(mysql, redis,rabbitmq and mongodb)

    Create a namespace
    create a StorageClass
    Create a StatefulSet for Roboshop DBs
    Single yaml file will have 
        statefulset - this will have all the details of replicasets, pods and at the end of the pod volumeMounts(PV) and volumeClaimTemplates(PVC) for claming the PV which got created dynamically using StorageClass
        headless service - with clusterIP: None
        general service for internal communications
        Remember to mount the DBs on the specific path 
            Redis --- mountPath: /data
            Mysql --- mountPath: /var/lib/mysql
            MongoDB --- mountPath: /data/db
            Rabbitmq --- mountPath: /var/lib/rabbitmq
    And Create the remaining services like Cart, Catalogue, frontend, payment, shipping, user etc
    And all these services will have configmap, service, hpa and deployment yaml - this file will have replicaset and pods


