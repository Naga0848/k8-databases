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

# Taints and Tolerations

    NodeSelector:-
        If we want, we can schedule a particular pod on to a specific node by using NodeSelector
        In the Pod defenition file we should use a NodeSelector and this NodeSelector is a label on that respective node(kubectl get nodes --show-labels)
        Here for this kind of pods, we use ImagePullPolicy 
            this is set to ImagePullPolicy: Always in a Pod
            This we have other than Always value, node cannot pull the images from the DockerHub if we did any changes to the code and built a new image
            Other ImagePullPolicies are IfNeverPresent and Never

    Understanding Taints and Tolerations:-
        Taints are applied to Nodes and Tolerations are applied to a Pod
        If we apply a taint to a node, we cannot schedule anything on to that node - basically we are polluting the node
        kunectl taint nodes <ip-address-of-node> key1=value1:NoSchedule
        Now, none of the pods get scheduled on to the above mentioned node.
        There are different conditions for a Node and basically these instructions are executed by the Scheduler.
        PreferNoSchedule - Informing Scheduler in a softway not schedule anything onto this node
        NoExecute - Existing pods on the node will be terminated
        NoSchedule - New pods will not be scheduled on to a node
        And incase if u want to schedule anything on this tained node, we should add tolerations to a pod parllel to the containers in a pod
            tolerations:
                - key: "key1"
                    operator: "Equal"
                    value: "value1"
                    effect: "NoExecute"
        Also, remember that we should add NodeSelector in the pod, if we want to schedule it on to a tained node

        
