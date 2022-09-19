# Creating Repository

aws ecr create-repository --repository-name eksdemoecr --region us-east-2

# Reppsitory Name
[YOUR-AWS-SUBSCRIPTION-ID].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr

# Login to ECR

aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin [YOUR-AWS-SUBSCRIPTION-ID].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr

# Tag the loal Docker Image to the Repository Name as follows

docker tag  eksdemoecr:v1 [YOUR-AWS-SUBSCRIPTION-ID].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr:v1

# Push the Image to ECR

docker push [YOUR-AWS-SUBSCRIPTION-ID].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr:v1

# Creating Cluster

# Use the Following URL for CloudFormation
- Go to the Cloud Formation and create a new Formation and use the following template URL
https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-06-10/amazon-eks-vpc-private-subnets.yaml


# The Cluster.json file will be as follows

````json
apiVersion: eksctl.io/v1alpha5 // version
kind: ClusterConfig // kind: the reason for which the file created, ClusterConfig, means created for EKS cluster creation
metadata:
  name: EKS-node-Cluster // cluster name
  region: us-east-2 // region where the cluster is available

vpc:
  id: vpc-03ecef376733dafd3  // VPM id received from stack of CloudFormation
  cidr: "192.168.0.0/16" // CIDR address ranges
  subnets: // public and private subntes for the internal and external communications
    public:
      us-east-2a: // 
        id: subnet-0a3ae750f8584f38c
      us-east-2b:
        id: subnet-073b5f4a67284ff9a
    private:
      us-east-2a:
        id: subnet-0aa44995415e64590
      us-east-2b:
        id: subnet-0e58f64081a75a947

nodeGroups: // confoguration for the nodes (VMs) where yhe app will be deployed
  - name: EKS-public-workers  // node name
    instanceType: t2.medium // the category of VM
    desiredCapacity: 2 // take 2 VMs
  - name: EKS-private-workers
    instanceType: t2.medium
    desiredCapacity: 1 // take 1
    privateNetworking: true

````

# CRaete a Cluster using the following command
- Command on windows
eksctl create cluster -f cluster.json --kubeconfig=c:\users\{user}\.kube\config
- {user} means the login user on the WIndows OS
- E.g.
    - If you are logged in as 'james' then command will be
        - eksctl create cluster -f cluster.json --kubeconfig=c:\users\james\.kube\config


- Command in Linux and Mac
eksctl create cluster -f cluster.json --kubeconfig=~/.kube/config

- Once the commd complete run the fllowinfg command to check if the cluster is created
     kubectl get svc

- sometimes its generates error for configuration, because the 
    configuration write failed in cluster
    - the error may be as follows
     - Unable to connect to the server: dial tcp: lookup 8189B04A666FE2962EDCE79097AC75C4.gr7.us-east-2.eks.amazonaws.com on 192.168.1.254:53: no such host

    - run the following command to resolve the configuration issues
        aws eks --region <NAME> update-kubeconfig --name <CLUSTER-NAME -FROM-JSON-FILE>
        - e.g.
            - aws eks --region us-east-2 update-kubeconfig --name EKS-node-Cluster

- above command will create 2 PODS which we can see using the folliewing command
    kubectl apply -f deploy.yaml      
    kubectl get deployments
- apply this file for configiuration
    kubectl apply -f service.yaml

- get services deployed 
    kubectl get services


- to access the applciation deployed on PODs first check pods  whcih will provde internal and external IP addresses and Ports for the communication
    kubectl get pods -o wide
- to get services on thse pods run the following command
    kucectl get services     
- to get the external IP address of the service with port run the following command
    kubectl get nodes -o wide
- to access the service from nodePort, add this port in the security group for input connection

- Finally set the inbound and outbound ports in sxecurity group to acees service
