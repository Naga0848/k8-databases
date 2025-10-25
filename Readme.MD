#### In this repo, we are understand about volumes and we are using statefulSets and creating RoboShop Databases(mysql, redis,rabbitmq and mongodb)

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


#### Volumes
#### Understanding PV, PVC and StorageClass (Elastic Block Storage)EBS and (Elastic File System)EFS

    EBS static provisioning
    =======================
        1. Install EBS drivers  kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.43  
            (should be installed on EKS)
        2. Provide access to EC2 instance through role, EBSCSIDriverPolicy
        3. create volume in same az as in EC2 instance  
        4. create PV, create PVC, create pod with nodeSelector option

    PV --> cluster level
    PVC --> namespace level

    EKS admins control cluster level objects...

    as a roboshop devops engineer, you need a disk to be created for your application,

    we will raise a ticket for this. storage (GB), filesystem type(ext4), etc..it is approved by roboshop team lead/delivery lead. Storage team also checks this and it is approved their team leader....

    then storage team creates the disk...

    provide these disk details to EKS admin, then they create PV for us and tell us the name.

    PV --> K8 resource, physical representation of the actual EBS storage.
    PVC --> it is the claim done by pods to mount the storage  (here we create the PVC yaml file, and if we want this storage to be attached to a pod, then we mention the pod definition file in the same yaml file)

    StorageClass --> k8 object used to create the volume dynamically...
        This is a cluster level object. If we have a storageclass yaml file, we can directly create the PVC and POD definition files in the other yaml file
        Understand that StorageClass yaml file is different and PVC and POD definition files can together.

#### If we attach the policy to a single node, it gets attached to all the 3 nodes, because we are using same IAM role across all the nodes
        EC2 >>> Security >> IAM Roles >> Add Permission >> Add the required policy
    
    EFS static provisioning
    =======================
        1. Install drivers  
            kubectl kustomize \
    "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-2.1" > private-ecr-driver.yaml
        2. Give permissions to nodes, EFSCSIDriverPolicy
        3. create EFS volume
        4. Allow port no 2049 in EFS SG from EC2 SG  (Go to SG of EFS > NFS, 2049 and now give the SG of EC2 nodes rather than giving IP an
        5. Create PV, PVC and mount to pod

    EBS vs EFS
    ===========
    1. EFS can be anywhere in network, EBS should be in same AZ as in EC2
    2. EBS can be accessed only by one instance at a time, EFS can be accessed by multiple instances
    3. EBS is faster than EFS.
    4. EBS is mainly used for OS and DB, EFS is used to store objects and files.
    5. EBS size is fixed, EFS is elastic it automatically grows upto 48TB.
    6. EBS filesystem we can select ext4, EFS filesystem is already decided by AWS as NFS
    7. EBS will not have any SG attached, but EFS is in network, so there will be SG attached and 2049 should be allowed.

#### Understanding Commands

    First we have to run the pv command i.e., kubectl apply for a PV
        Here u can see that a PV or EBS is created
    Second we have to run the pvc command i.e., kubectl apply for a PVC
        Here u can see that EBS is mounted to the POD, if we mentioned the pod definition file in the yaml file

#### Things to be remembered while creating EFS AND EBS

    when creating statically i.e., manually
        we need to create PV, PVC, POD and service(LB) in the same yaml file

        PV = physical storage = EBS Volume  (its a cluster level object)
        PVC = we are using this to claim a volume for our pod
        service = when we attached we volume to our pod, we have to access it right, as a result we are using the LB. And once everything got created, 
        please take the LB created and hit it in the browser and we get 403 Forbiden error as we dont have anything inside our volume
            Now, go to the location where your entire yaml file is loacted and perform the below commands to create a html file
                kubect exec -it nginx -- bash
                cd /usr/share/nginx/html 
                "<h1> Hi, i am from EBS Static </h1>" > index.html
                And when u hit the same browser with LB, we can see the ablove text.
                And if u try deleting the pod, we cannot acces the LB. gain, if  we create the pod, the EBS gets attached to the same pod
                Same happens with the EFSas well when provisioning statically

        Understand that we have to use the nodeselector if we want to place a pod on the specific Node
            For this, we have to see the labels which are on node  (kubectl get nodes -- labels)
            From this we have to look for a label with region and use the same label as a selector in the pod definition file

        When provisioning EBS Dynamically
            First create the StorageClass 
            And the PVC, POD and Service(LB) should be in a separate yaml file
            Here, storageClass will take care of creating the PV for us.
            Here Volume will be created by EKS ADMIN Team and we will make use of it by using our StorageClass
            The StorageClass is one time creation
            In PVC definition file, we have to mention the storageclass name and the disk will be attached to the desired pod.

        Steps for deleting
            Delete the PVC
            Delete PV
            Delete POD
            And finally the Volume(EBS or EFS)

        When provisioning EFS Dynamically (Here access points gets created, which are like EFS)the name of the accesspoint and PV will be same(please check)
            First create the StorageClass
            Create PVC, POD and Service in the different yaml file
            And finally, DATA will be accessed on the access points


        
#### Understanding  About Statefulsets

    StatefulSets are used only for the stateful applications --> DB

    Deployment --> stateless applications not for stateful applications
    Statefulset --> stateful applications.

    Deployment vs Statefulset
    ==========================
    1. Deployment is for stateless applications, Statefulset is for stateful applications
    2. PV and PVC is not mandatory for Deployment, but mandatory for stateful applications
    3. Statefulset need headless service... i.e no cluster IP
    4. Pods create in orderly manner in statefulset, Once first pod comes to running, then only other pod will create. While deletion reverse order follows
    5. Pod identities are preserved in statefulset, because if any pod crashes, statefulset create another pod with same name, so that communication is 
       easy between pods..


#### Understanding Taints and Tolerations

    NodeSelector:-
        If we want, we can schedule a particular pod on to a specific node by using NodeSelector
        In the Pod defenition file we should use a NodeSelector and this NodeSelector is a label on that respective node(kubectl get nodes --show-labels)
        Here for this kind of pods, we use ImagePullPolicy 
            ImagePullPolicy: Always is used in a Pod
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

        
