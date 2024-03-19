# Building and Deploying a Containerized Application with Amazon Elastic Kubernetes Service (Amazon EKS)
# overview
Kubernetes is an open-source system for automating the deployment, scaling, and management of containerized applications. Amazon Elastic Kubernetes Service (Amazon EKS) is a managed service that makes it easy for you to run Kubernetes on Amazon Web Services (AWS) without the need to maintain your own Kubernetes control plane. As part of the AnyCompany service team, you are responsible for deploying and maintaining the Amazon EKS and Kubernetes infrastructure.
In this Project, you install the command line interface (CLI) tools required to deploy and interact with Amazon EKS and Kubernetes. You then build an Amazon EKS cluster, node group, and deploy a sample application into your cluster.
# OBJECTIVES
⦁	Install and configure command line utilities for Amazon EKS.

⦁	Create and configure an Amazon EKS cluster and node group.

⦁	Deploy a sample application using Kubernetes manifests.

⦁	Review Kubernetes logs to troubleshoot an application.
# ENVIRONMENT
The following diagram shows the basic architecture of the existing environment
![Screenshot (17)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/edabecd1-df9a-4a33-b9cf-13c378ecefab)

*Image description: The preceding diagram depicts the infrastructure deployed already. It consists of a cluster of Kubernetes nodes spread across three availability zones. An AWS Cloud9 environment is provided to allow users to interact with the Kubernetes cluster*

The following list details the major resources in the diagram:

⦁	A Virtual Private Cloud (VPC) with 3 public subnets spread across 3 Availability Zones.

⦁	A Classic Load Balancer fronting 3 Kubernetes nodes in an Auto Scaling Group.

⦁	The Amazon EKS control plane managing the Kubernetes nodes.

⦁	An AWS Cloud9 environment.

SERVICES USED 

AWS Cloud9
AWS Cloud9 allows you to write, run, and debug your code with just a browser. With AWS Cloud9, you have immediate access to a rich code editor, integrated debugger, and built-in terminal with preconfigured AWS CLI. You can get started in minutes and no longer have to spend the time to install local applications or configure your development machine.

Elastic Kubernetes Service (Amazon EKS)
Amazon EKS is a managed service that makes it easy for you to use Kubernetes on AWS without needing to install and operate your own Kubernetes control plane.
# Task 1: Configure your AWS Cloud9 IDE
In this task, you connect to an AWS Cloud9 environment and disable its AWS-managed temporary credentials.

*AWS Cloud9 is a cloud-based integrated development environment (IDE) that you can use to write, run, and debug your code within your browser. It comes prepackaged with many tools that are commonly used in application development, including Docker, Python, and the AWS Command Line Interface (AWS CLI).*

⦁	You do not need the Cloud9 Welcome screen or any of the other default tabs that appear when you first launch AWS Cloud9, so choose the x next to each tab to close them.

⦁	At the top of the IDE, choose the  plus icon and select New Terminal.

Take a moment to familiarize yourself with the AWS Cloud9 IDE interface.

⦁	In the middle of the screen, a single terminal session is open in the editor. You can open multiple tabs in this window to edit files and run terminal commands.

⦁	The file navigator appears on the left side of the screen.

⦁	A gear icon appears on the right side of the screen. Choosing this icon opens the AWS Cloud9 Settings panel.

Every AWS Cloud9 workspace is automatically assigned IAM credentials that provide it with limited access to some AWS services in your account. We call these AWS managed temporary credentials. In this case, however, the AWS managed temporary credentials do not include all of the IAM permissions needed to complete this lab. In the following steps, you disable the AWS managed temporary credentials.

⦁	Choose the  gear icon at the top-right of the screen to open the AWS Cloud9 Preferences.

⦁	In the navigation panel on the left side of the screen, scroll down the page and choose AWS Settings.

⦁	Choose the  toggle button to disable AWS managed temporary credentials.

⦁	At the top of the screen, choose the x to close the Preferences tab and return to the tab containing your terminal session.

⦁	 Command: Enter the following command to confirm that the AWS managed temporary credentials have been successfully disabled.

aws sts get-caller-identity
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

Congratulations! You successfully connected to your AWS Cloud9 IDE.

# Task 2: Deploy an Amazon EKS cluster and managed node group
In this task, you use two utilities to bootstrap your Kubernetes application: kubectl and eksctl. Kubectl is a command-line tool that allows you to interact with resources inside your Kubernetes clusters. Among other things, you can use kubectl to deploy applications, create namespaces, scale deployments, and view logs. On the other hand, eksctl is a CLI tool for creating and managing clusters on EKS - Amazon’s managed Kubernetes service. While both tools enable you to work with Kubernetes clusters, they have different scopes and uses. Kubectl provides control over resources inside of a cluster, whereas eksctl helps you to manage the cluster itself.

Let’s start by installing kubectl.

⦁	 Command: To download the Kubernetes kubectl utility, enter the following command:

sudo curl --location -o /usr/local/bin/kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/linux/amd64/kubectl 

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 45.8M  100 45.8M    0     0  67.0M      0 --:--:-- --:--:-- --:--:-- 67.0M

⦁	 Command: To make kubectl an executable file, enter the following command:

sudo chmod +x /usr/local/bin/kubectl
 

Expected output:

None, unless there is an error.

⦁	 Command: To display the version of the kubectl utility and verify it is installed properly, enter the following command:

kubectl version --output=yaml --client


Expected output: The output should look similar to this:

Explain

clientVersion:
 
  buildDate: "2023-04-15T00:36:29Z"
 
  compiler: gc
 
  gitCommit: 4a3479673cb6d9b63f1c69a67b57de30a4d9b781
  
  gitTreeState: clean
  
  gitVersion: v1.26.4-eks-0a21954
 
  goVersion: go1.19.8
 
  major: "1"
 
  minor: 26+
 
  platform: linux/amd64

kustomizeVersion: v4.5.7

# Next, you install the eksctl command line utility.

⦁	 Command: To download and unzip eksctl, enter the following command:

curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 32.9M  100 32.9M    0     0  19.8M      0  0:00:01  0:00:01 --:--:-- 23.9M

⦁	 Command: To move eksctl to the /usr/local/bin directory enter the following command:

sudo mv -v /tmp/eksctl /usr/local/bin
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

‘/tmp/eksctl’ -> ‘/usr/local/bin/eksctl’

 Note: By moving the downloaded file to the /usr/local/bin folder, you can call the eksctl command without including its full filepath because the directory is already in your PATH.

⦁	 Command: To verify eksctl is installed, enter the following command:

eksctl version
 
 Expected output: The output displays the version of eksctl, which should be at least version 0.153.0.

Now that you installed the utilities required to create and manage Amazon EKS and Kubernetes, you create your first Amazon EKS cluster.

⦁	 Command: Start by saving the region you are working in and your account ID to shell variables. Enter the following command:

TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
export AWS_REGION=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
echo "Your AWS Account ID is $ACCOUNT_ID and you are working in the $AWS_REGION region"

Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

Your AWS Account ID is 111111111111 and you are working in the us-west-2 region

⦁	 Command: Enter the following command to deploy an Amazon EKS cluster called eks-lab-cluster, along with a managed node group:
Explain

eksctl create cluster \

--name eks-lab-cluster \

--nodegroup-name worknodes-1 \

--node-type t3.medium \

--nodes 2 \

--nodes-min 1 \

--nodes-max 3 \

--managed \

--version 1.26 \

--region ${AWS_REGION}

**Learn more:** When you create an Amazon EKS cluster using eksctl, the utility builds a new Virtual Private Cloud (VPC), three public subnets, and three new private subnets for use with the Amazon EKS control plane cluster and nodes. However, there are additional command flags you can use to specify the use of an existing VPC and subnets. 

The following list describes each flag used on the command you just entered:

⦁	eksctl create cluster: Creates a new Amazon EKS cluster.

⦁	–name eks-lab-cluster: Specifies a name for the Amazon EKS cluster.

⦁	–nodegroup-name worknodes-1: Specifies a name for the Amazon EKS node group.

⦁	–node-type t3.medium: Designates the instance type used for the worker nodes.

⦁	–nodes 2: Sets the desired number of worker nodes in the Auto Scaling configuration.

⦁	–nodes-min 1: Sets the minimum number of worker nodes Auto Scaling configuration.

⦁	–nodes-max 3: Sets the maximum number of worker nodes in the Auto Scaling configuration.

⦁	–managed: Creates a fully managed node group. For more information, refer to the Additional resources section at the end of the lab.

⦁	–version 1.26: Specifies the version of Kubernetes to deploy in the Amazon EKS cluster.

⦁	–region ${AWS_REGION}: Specifies the region into which the cluster and node group will be deployed.
 
Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

2023-11-02 20:34:21 [ℹ]  eksctl version 0.164.0

2023-11-02 20:34:21 [ℹ]  using region us-west-2

2023-11-02 20:34:21 [ℹ]  setting availability zones to [us-west-2d us-west-2a us-west-2c]

2023-11-02 20:34:21 [ℹ]  subnets for us-west-2d - public:192.168.0.0/19 private:192.168.96.0/19

2023-11-02 20:34:21 [ℹ]  subnets for us-west-2a - public:192.168.32.0/19 private:192.168.128.0/19

2023-11-02 20:34:21 [ℹ]  subnets for us-west-2c - public:192.168.64.0/19 private:192.168.160.0/19

2023-11-02 20:34:21 [ℹ]  nodegroup "worknodes-1" will use "" [AmazonLinux2/1.26]

2023-11-02 20:34:21 [ℹ]  using Kubernetes version 1.26

2023-11-02 20:34:21 [ℹ]  creating EKS cluster "eks-lab-cluster" in "us-west-2" region with managed nodes

2023-11-02 20:34:21 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup

2023-11-02 20:34:21 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-west-2 --cluster=eks-lab-cluster'

2023-11-02 20:34:21 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "eks-lab-cluster" in "us-west-2"

2023-11-02 20:34:21 [ℹ]  CloudWatch logging will not be enabled for cluster "eks-lab-cluster" in "us-west-2"

2023-11-02 20:34:21 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-west-2 --cluster=eks-lab-cluster'

2023-11-02 20:34:21 [ℹ]  

2 sequential tasks: { create cluster control plane "eks-lab-cluster", 
    2 sequential sub-tasks: { 
        wait for control plane to become ready,
        create managed nodegroup "worknodes-1",
    } 
}

2023-11-02 20:34:21 [ℹ]  building cluster stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:34:22 [ℹ]  deploying stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:34:52 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:35:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:36:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:37:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:38:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:39:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:40:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:41:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:42:22 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-cluster"

2023-11-02 20:44:23 [ℹ]  building managed nodegroup stack "eksctl-eks-lab-cluster-nodegroup-worknodes-1"

2023-11-02 20:44:24 [ℹ]  deploying stack "eksctl-eks-lab-cluster-nodegroup-worknodes-1"

2023-11-02 20:44:24 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-nodegroup-worknodes-1"

2023-11-02 20:44:54 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-nodegroup-worknodes-1"

2023-11-02 20:45:39 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-nodegroup-worknodes-1"

2023-11-02 20:47:13 [ℹ]  waiting for CloudFormation stack "eksctl-eks-lab-cluster-nodegroup-worknodes-1"

2023-11-02 20:47:13 [ℹ]  waiting for the control plane to become ready

2023-11-02 20:47:14 [!]  unable to write kubeconfig , please retry with 'eksctl utils write-kubeconfig -n eks-lab-cluster': failed to obtain exclusive lock on kubeconfig lockfile: open /tmp/00d67397.eksctl.lock: permission denied

2023-11-02 20:47:14 [ℹ]  no tasks

2023-11-02 20:47:14 [✔]  all EKS cluster resources for "eks-lab-cluster" have been created

2023-11-02 20:47:14 [ℹ]  nodegroup "worknodes-1" has 2 node(s)

2023-11-02 20:47:14 [ℹ]  node "ip-192-168-20-241.us-west-2.compute.internal" is ready

2023-11-02 20:47:14 [ℹ]  node "ip-192-168-67-137.us-west-2.compute.internal" is ready

2023-11-02 20:47:14 [ℹ]  waiting for at least 1 node(s) to become ready in "worknodes-1"

2023-11-02 20:47:14 [ℹ]  nodegroup "worknodes-1" has 2 node(s)

2023-11-02 20:47:14 [ℹ]  node "ip-192-168-20-241.us-west-2.compute.internal" is ready

2023-11-02 20:47:14 [ℹ]  node "ip-192-168-67-137.us-west-2.compute.internal" is ready

2023-11-02 20:47:14 [✔]  EKS cluster "eks-lab-cluster" in "us-west-2" region is ready

⦁	To stop creating the new cluster, press Ctrl+C.

 Expected output:
None, unless there is an error.

If you had allowed the new cluster to finish building, eksctl would have saved a configuration file to ~/.kube/config. This file is commonly referred to as the kubeconfig and, among other things, contains the authentication credentials used to connect to the cluster. In order to connect to the prebuilt cluster, you will use a kubeconfig saved to your AWS Cloud9 IDE.

⦁	 Command: Enter the following command to view the contents of the kubeconfig file:

cat /home/ec2-user/environment/scripts/config
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJQWt4TEQra2ZOWnN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBNE1UZ3hOelUxTXpoYUZ3MHpNekE0TVRVeE56VTFNemhhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURBWnNFMC8wZ1ptWlhKNWhpV2wwYk53bU5zK01EbCtIQnk5amhEREE0UnZSZmhHN1hvZUJlTC9qZjkKRDVHRjhKYzdqNEI2RTM4YlJidmZzNmtKRWxLQ21ISnU2aGx0OEZRdzBkVE5QMWQ3QU9BTWdxMWhpRms1aXJISgprMVFLd0F3M0ZYcXIrdC9IYXN2OVl1SE1pU0E2dGJUKzBCQ285aFVuaDk0KzFxcCtUZHFMN3hSWE5MeldLejdkCjZoWmtLVk1iUlM1UWo4cGRRNnFwcEg1WUtiUStUL1VsQkNrMlNqb0xNeFlKS0xnbXpmYTNRUHdxMjNMaC9TbnEKbk5EL3JMd0k5VXYxQlFyUi9DVys3cDJKV1dyaUdXYVdPTDhwTHpZT004bzdjMEkvY0dYOEZ4WG9pNHZ5amZtaQpUK3YrTnhyZzlpb1VSNDR6bkJTWG5jR3U0TTR4QWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJSVytDWlZCVWFoSWIwY2R4TG5oRGl1OWp2QjhEQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQngxVXY5SC9zUQo0dk00ZTYzN0UrMHprWDJkbCtFa2RjOGhtWmlRK1o4eTl2ZGVLWktwZk9DT2RGaG9yM2UyS2l2TTR0TWtJM3VICmVMa3F0K09ZTnZuK0doR04zbTRpOWpNQW5nYVNDcCtEa0lKV21pZ2cweTJjajNRSkxxL3VBYk1ZTWVWRmFZVDIKd1V6VGZEcjVVR0NMZ2JYU09QRm1ZSWZZd3ZNWmZOQTBGdTl2M3lraCtlb2RaVWIzNnVocExmWktDREhFeHg4UgovK1F0ZWJGTFBMVldZclMvYVdNOFhNb2ROTGhlRmpwdm5YSkxWd3pFYWRoMUVTd2xyWUlWdGs2OGFoWDU3cEtLCnYxMGt6T2JQZkh3K2ZPNDNMemV2VmdMeGNHSm5jVEl3aVYzNXZCc0dGVHNQSmpNeVQ0VVdSSG5UMU91RjlTV3YKQzFJclpTNlUxU3p6Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://5362FF44B6D8CC4262509F4A015546B6.gr7.us-west-2.eks.amazonaws.com
  name: dev-cluster.us-west-2.eksctl.io
contexts:
- context:
    cluster: dev-cluster.us-west-2.eksctl.io
    user: i-09201a6f67c122693@dev-cluster.us-west-2.eksctl.io
  name: i-09201a6f67c122693@dev-cluster.us-west-2.eksctl.io
current-context: i-09201a6f67c122693@dev-cluster.us-west-2.eksctl.io
kind: Config
preferences: {}
users:
- name: i-09201a6f67c122693@dev-cluster.us-west-2.eksctl.io
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - eks
      - get-token
      - --cluster-name
      - dev-cluster
      - --region
      - us-west-2
      command: aws
      env:
      - name: AWS_STS_REGIONAL_ENDPOINTS
        value: regional
      provideClusterInfo: false

The following list explains key information contained in the kubeconfig file:

⦁	apiVersion: This field indicates the version of the Kubernetes API that the kubeconfig file is written in.

⦁	clusters: This section contains information about the Kubernetes clusters that the kubeconfig file can be used to access. In this case, there is only one cluster, dev-cluster.us-west-2.eksctl.io.

⦁	certificate-authority-data: This field contains the certificate authority data for the cluster, which is used to verify the server’s certificate.

⦁	current-context: This field indicates the current context, which is the context that commands will be run against by default.

⦁	users: This section contains information about the users that can be used to authenticate to the clusters. In this case, is only one user, i-09201a6f67c122693@dev-cluster.us-west-2.eksctl.io.

⦁	exec: This field contains a command to be executed to fetch user credentials. In this case, the command is 
aws eks get-token
, which fetches a token for the specified cluster and region.

⦁	 Command: Enter the following command to move the kubeconfig file into your home directory:
mv /home/ec2-user/environment/scripts/config ~/.kube/config

 Expected output:
None, unless there is an error.

Now that you’ve moved the kubeconfig file into your home directory, you should be able to connect to the cluster. Let’s start by finding the cluster name.

⦁	 Command: Enter the following command to retrieve the cluster name:
eksctl get cluster --region $AWS_REGION

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

NAME            REGION          EKSCTL CREATED
dev-cluster     us-west-2       True

 Note: The eks-lab-cluster you started creating in the preceding steps may also appear in the output. As you continue through this lab, only use the cluster called dev-cluster.

⦁	 Command: Now that we’ve found the cluster name, let’s find the associated nodegroup:
eksctl get nodegroup --cluster=dev-cluster --region $AWS_REGION

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

CLUSTER         NODEGROUP       STATUS  CREATED                 MIN SIZE        MAX SIZE        DESIRED CAPACITY        INSTANCE TYPE   IMAGE ID        ASG NAME                        TYPE
dev-cluster     dev-nodes       ACTIVE  2023-09-05T14:03:02Z    2               4               3                       t3.medium       AL2_x86_64      eks-dev-nodes-f2c532d3-aafd-02fc-3d87-9aeff059d5e0       managed

The nodegeroup is called dev-nodes and has a desired capacity of 3 nodes. For the purposes of our project, this is more nodes than we require. Let’s reduce the desired node count to 2.

⦁	 Command: Enter the following command to reduce the desired node count to 2:
eksctl scale nodegroup --cluster=dev-cluster --nodes=2 --name=dev-nodes --region $AWS_REGION

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

2023-09-07 20:29:40 [ℹ]  scaling nodegroup "dev-nodes" in cluster dev-cluster

2023-09-07 20:29:41 [ℹ]  initiated scaling of nodegroup

2023-09-07 20:29:41 [ℹ]  to see the status of the scaling run `eksctl get nodegroup --cluster dev-cluster --region us-west-2 --name dev-nodes`

# Well done! Now that we’ve seen how eksctl can be used to create and update a cluster, let’s confirm that kubectl is able to communicate with resources inside the cluster.

⦁	 Command: Enter the following command to retrieve the cluster state:
kubectl cluster-info

 Expected output: The output should be similar to the following:

Kubernetes control plane is running at https://748E381CD720F24CE6AA7B647A8B8A81.gr7.us-west-2.eks.amazonaws.com

CoreDNS is running at https://748E381CD720F24CE6AA7B647A8B8A81.gr7.us-west-2.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

The output returns two URLs. The first points to the managed cluster control plane and the second points to a DNS proxy running inside the cluster. This confirms that the cluster is up and running and that kubectl can connect to the resources inside of it.
 
 Consider: How did kubectl connect to the cluster? Amazon EKS uses IAM to provide authentication to your Kubernetes cluster, but it still relies on native Kubernetes Role Based Access Control (RBAC) for authorization. This means that IAM is only used for authentication of valid IAM entities. All permissions for interacting with your Amazon EKS cluster’s Kubernetes API is managed through the native Kubernetes RBAC system.
# Congratulations! You successfully installed eksctl and kubectl and confirmed that the dev-cluster is ready to host your application.
# Task 3: Deploy and configure a sample application
In this task, you deploy an application into your Amazon EKS cluster. The application is a corporate employee directory that uses a standard three-tier architecture consisting of:

⦁	A containerized python frontend

⦁	A containerized backend API written in dotnet

⦁	An Amazon DynamoDB database

The frontend and backend are created as separate Kubernetes deployments, each consisting of a replicaset with two pods. Both deployments are exposed using Kubernetes services. The following diagram depicts the cluster architecture that will be used to deploy your application:

![Screenshot (18)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/38ed9219-65f6-47df-b8da-7905182292db)

Image description: In the preceding diagram, the Kubernetes application architecture is depicted. It consists of two deployments, each containing two pods in replicasets. Each deployment is fronted by a service.

This image was created using Trois-Six k8s-diagrams

Rather than storing the employee data inside the cluster, your application uses an Amazon DynamoDB database. Start by creating the database.
# TASK 3.1: CREATE THE DATABASE
Create an Amazon DynamoDB database to store your employee data.

⦁	 Command: Enter the following command to create an Amazon DynamoDB database for the application:
aws dynamodb create-table --table-name Employees --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "id",
                "AttributeType": "S"
            }
        ],
        "TableName": "Employees",
        "KeySchema": [
            {
                "AttributeName": "id",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "CREATING",
        "CreationDateTime": "2023-08-24T15:25:32.247000+00:00",
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 1,
            "WriteCapacityUnits": 1
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-west-2:111111111111:table/Employees",
        "TableId": "bb3ea3b6-b1fb-4756-83a1-f16ef6f0145b",
        "DeletionProtectionEnabled": false
    }
}

The following list describes the commands and flags used on the command:

⦁	aws dynamodb create-table --table-name Employees: Creates a new table called Employees.

⦁	–attribute-definitions AttributeName=id,AttributeType=S: Creates a single attribute named id with a string type (S).

⦁	–key-schema AttributeName=id,KeyType=HASH: Defines the key schema for the table. The attribute id is set as the HASH key type, which is the primary key of the table.

⦁	–provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1: The provisioned throughput for the table is set to 1 read capacity unit and 1 write capacity unit. This determines the maximum amount of read and write operations that can be performed on the table per second.
# TASK 3.2: DEPLOY THE APPLICATION FRONTEND
Now that your database has been created, let’s take a look at the default resources that have been deployed into your cluster.

⦁	 Enter the following command to view the deployments, services, and pods currently running in the cluster:
kubectl get deployment,service,pod --all-namespaces

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           21m

NAMESPACE     NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes   ClusterIP   172.20.0.1    <none>        443/TCP         21m
kube-system   service/kube-dns     ClusterIP   172.20.0.10   <none>        53/UDP,53/TCP   21m

NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   pod/aws-node-cdrs6             1/1     Running   0          21m
kube-system   pod/aws-node-r9lf9             1/1     Running   0          21m
kube-system   pod/coredns-6c45d94f67-9sd7b   1/1     Running   0          19m
kube-system   pod/coredns-6c45d94f67-pqswn   1/1     Running   0          19m
kube-system   pod/kube-proxy-jnmj2           1/1     Running   0          21m
kube-system   pod/kube-proxy-w5bpf           1/1     Running   0          21m
 
 Note: The kubectl get [TYPE] [NAME] [NAMEPSACE] command returns one or more resources in a table format. In this case, kubectl get deployment,service,pod --all-namespaces returned a deployment for the DNS proxy, two services enabling pod-to-pod communication, and 6 pods in the kube-system namespace.

⦁	Deployments can be thought of as the blueprint for an application.

⦁	Pods are the smallest units of compute that can be created and managed in Kubernetes.

⦁	Services allow network access to one or more pods.

⦁	Namespaces are logical constructs that are used for grouping and isolating objects within a cluster.

Now that you’ve seen what’s already in your cluster, it’s time to deploy your application using Kubernetes manifests.

 Note: Kubernetes manifests contain key-value pairs defining an application’s desired state. They can be written in YAML or JSON and are an example of declarative programming. As opposed to imperative programming which explicitly states how each instruction is to be performed, declarative programming describes a desired application state. The declarative programming language then deploys the logic needed to achieve the desired state.

Start by creating a deployment manifest for the application’s web frontend.

 Note: The source code for the application has been saved to an S3 bucket.

⦁	 To create a shell variable to store the S3 URL to the front end code location:

⦁	Replace the FRONT_END_SOURCE_CODE_URL placeholder value with the FrontendSourceCodeURL value that is listed to the left of these instructions.

FRONTEND_S3=FRONT_END_SOURCE_CODE_URL

 Expected output:
None, unless there is an error.

Next, create a manifest that retrieves the source code from S3, builds a container for the frontend application, and then defines a Kubernetes deployment that the container will run in.

⦁	 Command: Enter the following command to create a deployment manifest:

Explain
cat <<EOF > ~/environment/deployment-frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: frontend
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - image: python:3.10-slim
        name: frontend
        env:
          - name: API_ENDPOINT
            value: "http://backend.default.svc.cluster.local"
          - name: FLASK_APP
            value: "application.py"
        command: ["/bin/sh","-c"]
        args: 
         - mkdir /app ; 
           cd /app ;  
           apt update ; 
           apt install wget -y curl unzip ;
           curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" ;
           unzip awscliv2.zip ;
           ./aws/install ;
           aws s3 cp $FRONTEND_S3 . ;
           unzip DirectoryFrontend.zip ;
           cd DirectoryFrontend/ ;
           pip3 install -r requirements.txt ;
           flask run --host 0.0.0.0 --port 80

EOF
 
 Expected output:
None, unless there is an error.

 Note: This Kubernetes manifest defines a deployment that manages two replicas of a container running a Python application. The container uses the Python 3.10-slim image and is labeled as ‘frontend’. The manifest also specifies environment variables for the application and a series of commands and arguments to set up and run the application

 Consider: Take a moment to review the structure of the manifest file and note the following:

⦁	apiVersion: apps/v1 and kind: Deployment: These lines specify the version of the Kubernetes API and the type of resource to create, in this case, a deployment.

⦁	metadata: This section contains data that is used to help identify objects inside the cluster. In this case, it assigns frontend as a name and a label.

⦁	spec: This section defines the desired state of the deployment.

⦁	replicas: 2: This line specifies that two instances of the application should be running.

⦁	selector: This is used to identify the pods managed by the deployment.

⦁	containers: This section states that a python:3.10-slim image should be used for the container and that it should be named frontend.

⦁	env: Sets environment variables for the container, including the API endpoint.

⦁	args: Specifies the arguments to pass to the container. In this case, it sets up the application environment, downloads and extracts the application code, installs the required Python packages, and runs the Flask application.

Now that you’ve created the frontend manifest, it’s time to deploy it into your cluster.

⦁	 Command: Enter the following command to create the frontend deployment:
kubectl apply -f ~/environment/deployment-frontend.yaml
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

deployment.apps/frontend created

 Note: The kubectl apply command is used to create and update Kubernetes resources.

It looks like the frontend deployed successfully, but before you can test it, it needs a service that enables you to connect to its web interface.

⦁	 Enter the following command to create a second manifest defining the state for your frontend service:

Explain
cat <<EOF > ~/environment/service-frontend.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: frontend
  name: frontend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: frontend
  type: LoadBalancer

EOF
 
 Note: This manifest creates a service called frontend, targeting port 80 on any pod with the app: frontend label. Let’s take a closer look at the manifest components:

⦁	apiVersion: apps/v1 and kind: Service: Services are used to expose a logical set of pods as a network service.

⦁	metadata: Similar to the deployment manifest, the metadata field is used to assign a name and label to the service.

⦁	spec: The service is set to accept HTTP traffic on port 80.

⦁	selector: The selector acts as a filter. In this case, it will only forward requests on port 80 that include the app: frontend label.

⦁	 Command: Enter the following command to use the kubectl apply command to create the frontend service:
kubectl apply -f ~/environment/service-frontend.yaml

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

service/frontend created

# Great job. You’ve successfully launched the application frontend deployment and service. Let’s confirm that everything is in a Running state.
⦁	 Command: Enter the following command to view the status of the resources you deployed:
 
  kubectl get deployment,service,pod

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend   2/2     2            2           62m

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)        AGE
service/frontend     LoadBalancer   172.20.5.236   aa342c97ec5f6499280c38cec2c5a4b0-111111111111.us-west-2.elb.amazonaws.com   80:32742/TCP   19m
service/kubernetes   ClusterIP      172.20.0.1     <none>                                                                    443/TCP        152m

NAME                           READY   STATUS    RESTARTS   AGE
pod/frontend-86454f8fb-5nd8r   1/1     Running   0          62m
pod/frontend-86454f8fb-nplm5   1/1     Running   0          62m

 Note: The output indicates that two replicas of the frontend application are in a Running state and fronted by a load balancer. If the status is not Running, verify that you entered the previous commands correctly and repeat them if necessary.

⦁	Copy the External IP and paste it into a new web browser tab. Be sure to prepend the LoadBalancer URL with 
http://
, as many common browsers default to using https://. If the webpage fails to load, the load balancer may not yet be fully online. Wait a couple of minutes and then refresh the page.

The value you are looking for should look similar to this:

aa342c97ec5f6499280c38cec2c5a4b0-111111111111.us-west-2.elb.amazonaws.com

The web page opens and displays an API error message.

![Screenshot (19)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/ee1f36ac-f57d-4208-9a81-02b4cd3f773b)

Image description: In the preceding image, the application frontend is shown and it displays an error message stating that an API exception occurred.
 
 Note: If the website does not load correctly and display the API error message, wait 2–3 minutes and refresh the page. The load balancer can take some time to come online fully. If the website still does not load, verify that the frontend application state is Running.

 Note: You haven’t yet deployed the application backend or connected it to the database, so the exception is expected.

# TASK 3.3: DEPLOY THE APPLICATION BACKEND
In the following steps, you deploy the application backend.

⦁	Return to your AWS Cloud9 terminal.

 Note: Just like the source code for the application frontend, the code for the backend has also been saved to an S3 bucket.

⦁	 To create a shell variable to store the S3 URL to the front end code location:

⦁	Replace the BACK_END_SOURCE_CODE_URL placeholder value with the BackendSourceCodeURL value
 
 Expected output:
None, unless there is an error.

Next, create a manifest that creates a deployment and launches the backend source code in a container.

⦁	 Command: Enter the following command to create a deployment manifest:

Explain
cat <<EOF > ~/environment/deployment-backend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: backend
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - image: mcr.microsoft.com/dotnet/sdk:6.0
        env:
        - name: ASPNETCORE_URLS
          value: "http://+:80"
        command: ["/bin/sh","-c"]
        args: 
         - mkdir /app ; 
           cd /app ;
           apt update ;
           apt install wget -y curl unzip ;
           curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" ;
           unzip awscliv2.zip ;
           ./aws/install ;
           aws s3 cp $BACKEND_S3 . ;
           unzip DirectoryBackend.zip ;
           cd DirectoryBackend/ ;
           dotnet run
        name: backend
        resources: {}
EOF

 Note: This manifest creates a deployment with two replicas called backend. Each replica will run a dotnet application inside of a container.

Let’s take a closer look at the manifest components:

⦁	apiVersion: apps/v1 and kind: Deployment: These lines specify the version of the Kubernetes API and the type of resource to create, in this case, a deployment.

⦁	metadata: This section contains data that is used to help identify objects inside the cluster. In this case, it assigns backend as a name and a label.

⦁	spec: This section defines the desired state of the deployment.

⦁	replicas: 2: This line specifies that two instances of the application should be running.

⦁	selector: This is used to identify the pods managed by the deployment.

⦁	containers: This section states that a mcr.microsoft.com/dotnet/sdk:6.0 image should be used for the container and that it should be named backend.

⦁	env: Sets environment variables for the container, including the ASP.NET Core URLs.

⦁	args: Specifies the arguments to pass to the container. In this case, it sets up the application environment, downloads and extracts the application code, and runs the .NET application.

Even though the backend doesn’t have a web interface that users will connect to, it still needs a service enabling it to communicate with the frontend and your Amazon DynamoDB database.

⦁	 Command: Enter the following command to create a service manifest:

Explain
cat <<EOF > service-backend.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
   -  protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
EOF
 Expected output:
None, unless there is an error.
⦁	 Command: Enter the following command to deploy the backend deployment and service:
kubectl apply -f service-backend.yaml,deployment-backend.yaml
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

deployment.apps/backend created

service/backend created

Take a minute to review the resources deployed into your cluster.

⦁	 Command: Enter the following command to display information about all of the resources you’ve deployed into your cluster:

kubectl get all
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

NAME                           READY   STATUS    RESTARTS   AGE
pod/backend-65946ddc59-6kjhr   1/1     Running   0          3m31s
pod/backend-65946ddc59-x7jvp   1/1     Running   0          3m31s
pod/frontend-7f49fffc5-w89d9   1/1     Running   0          18m11s
pod/frontend-7f49fffc5-x6bfd   1/1     Running   0          18m11s

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)        AGE
service/backend      ClusterIP      172.20.38.104    <none>                                                                    80/TCP         3m31s
service/frontend     LoadBalancer   172.20.130.197   aa342c97ec5f6499280c38cec2c5a4b0-111111111111.us-west-2.elb.amazonaws.com   80:32007/TCP   17m44s
service/kubernetes   ClusterIP      172.20.0.1       <none>                                                                    443/TCP        28m22s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/backend    2/2     2            2           3m31s
deployment.apps/frontend   2/2     2            2           18m11s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/backend-65946ddc59   2         2         2       3m31s
replicaset.apps/frontend-7f49fffc5   2         2         2       18m11s

 Knowledge check: Recall that at the beginning of this project, you saw that a variety of default resources, such as a DNS proxy, had been deployed into your cluster. Why weren’t those resources included in the command output?
# Solution
So far, so good. Both deployments show that they are up-to-date and have 2 available pods running in ReplicaSets.
# Congratulations! You have deployed the application frontend.
# Task 4: Troubleshoot the application
In this task, you explore the application and discover that it is not functioning properly. You use the kubectl utility to view logs and troubleshooting the application.
# TASK 4.1: IDENTIFY THE PROBLEM
⦁	 Command: To find the frontend load balancer address, enter the following command:
kubectl get service frontend --output wide

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

NAME       TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)        AGE   SELECTOR
frontend   LoadBalancer   172.20.12.31   aa342c97ec5f6499280c38cec2c5a4b0-111111111111.us-west-2.elb.amazonaws.com   80:32625/TCP   24m   app=frontend

 Note: Including the –output wide flag makes kubectl provide additional information about the service. This can be useful when you require a more comprehensive understanding of the service’s configuration and status.

⦁	Copy the External IP and paste it into a new web browser tab. Be sure to prepend the LoadBalancer URL with 
http://
.

 Caution: If the webpage fails to load, the backend load balancer may not yet be fully online. Wait a couple of minutes and then refresh the page.

The application launches a home page displaying an empty employee directory

Image description: In the preceding image, the application displays an empty employee directory. 

![Screenshot (20)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/ff212988-78de-40ec-8b2d-69ec81cef8ac)

 Beneath the directory table are two strings showing the frontend pod ID and the service ID.

 Note: Recall that the kubectl get all command showed that there were two frontend and two backend pods. In this case, it means that there are two instances of each container running in your cluster.

⦁	Refresh the page and note the ID displayed next to Frontend → Hello from and Service → Hello from. These IDs correspond to the pod names in your cluster. Each time you refresh the page, these values are updated to show the frontend pod you are connected to and the backend pod with which it is communicating.

Let’s add an employee to the directory.

⦁	Choose the Add button at the top of the application page.

You are brought to the Corporate Directory - Add/Edit page.

⦁	Enter the following details to create a new user:

⦁	For Full Name, enter 
Arnav Desai
.
⦁	For Location, enter 
Las Vegas
.
⦁	For Job Title, enter 
DevOps Engineer
.
⦁	Choose the checkbox next to Linux User.

⦁	Scroll to the bottom of the page and select Save.

Uh-oh! The application returns another API exception. It looks like we’ll need to dig into the logs to figure out what’s going on. Since the error only appeared after you deployed the backend, let’s start by examining those logs.

⦁	Leave the browser tab connected to the application open and return to your AWS Cloud9 terminal.

⦁	 Command: Enter the following command to view logs from the backend deployment:

kubectl logs deployment/backend | head

 Note: Kubernetes resources often generate large amounts of logging data. In this case, you have piped the output into the head command to ensure that the logs at the start of the output are easily identified.
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

Found 2 pods, using pod/backend-6d48b67bf6-bptxz

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Get:1 http://deb.debian.org/debian bullseye InRelease [116 kB]

Get:2 http://deb.debian.org/debian-security bullseye-security InRelease [48.4 kB]

Get:3 http://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]

Get:4 http://deb.debian.org/debian bullseye/main amd64 Packages [8183 kB]

Get:5 http://deb.debian.org/debian-security bullseye-security/main amd64 Packages [245 kB]

Get:6 http://deb.debian.org/debian bullseye-updates/main amd64 Packages [17.5 kB]

Fetched 8653 kB in 2s (4252 kB/s)
 
 Caution: Look at the first line of the log output. It states that the backend deployment contains two pods, but that the command only returns logs from one of the pods. This is a problem, because if the other pod caused the exception, the error logs will not appear in the output. To retrieve logs from all pods in the backend deployment, you can modify the command to retrieve from all pods that include the label app=backend.

⦁	 Command: Enter the following modified command to stream logs from all pods in the backend deployment and filter them to only show lines that include the word exception:

kubectl logs -l app=backend -f | grep -i exception

The following list describes each flag used on the command you just entered:

⦁	kubectl logs: Prints pod logs to standard out.

⦁	-l app=backend: Specifies that logs should be returned for all pods with the app=backend label.

⦁	-f: Streams the output.

⦁	grep -i exception: Filters results for log entries that include the string exception.
 
 Expected output:

A blank line in your terminal appears.

Now that you are streaming the logs from your application, you intentionally provoke an error and then review the logs it generates.

⦁	Return to the browser tab connected to the application and choose the Back button to go back to the Corporate Directory - Add/Edit page.

⦁	Attempt to create a user again. Enter the following details:

⦁	For Full Name, enter 
Arnav Desai
.
⦁	For Location, enter 
Las Vegas
.
⦁	For Job Title, enter 
DevOps Engineer
.
⦁	Choose the checkbox next to Linux User.

⦁	Scroll to the bottom of the page and select Save.

As expected, the application returns an error message.

⦁	Return to the AWS Cloud9 terminal and review the logs.
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

      Connection id "0HMTE4E4DKVCA", Request id "0HMTE4E4DKVCA:00000002": An unhandled exception was thrown by the application.
      Amazon.DynamoDBv2.AmazonDynamoDBException: User: arn:aws:sts::111111111111:assumed-role/EksNodeRole/i-00e1b46173902029e is not authorized to perform: dynamodb:PutItem on resource: arn:aws:dynamodb:us-west-2:111111111111:table/Employees because no identity-based policy allows the dynamodb:PutItem action
       ---> Amazon.Runtime.Internal.HttpErrorResponseException: Exception of type 'Amazon.Runtime.Internal.HttpErrorResponseException' was thrown.
         --- End of inner exception stack trace ---
         at Amazon.Runtime.Internal.HttpErrorResponseExceptionHandler.HandleExceptionStream(IRequestContext requestContext, IWebResponseData httpErrorResponse, HttpErrorResponseException exception, Stream responseStream)
         at Amazon.Runtime.Internal.HttpErrorResponseExceptionHandler.HandleExceptionAsync(IExecutionContext executionContext, HttpErrorResponseException exception)
         at Amazon.Runtime.Internal.ExceptionHandler`1.HandleAsync(IExecutionContext executionContext, Exception exception)
         at Amazon.Runtime.Internal.ErrorHandler.ProcessExceptionAsync(IExecutionContext executionContext, Exception exception)

 Note: The error message indicates that the application is attempting to use an assumed role called EksNodeRole to add your user to the database. This is the IAM role currently assigned to the nodes in your cluster. However, the role “is not authorized to perform: dynamodb:PutItem”.

You could resolve this error by updating the EksNodeRole to include Amazon DynamoDB permissions, however using a Kubernetes service account is generally considered a best practice. Among the advantages offered by service accounts are the following:

⦁	Principle of Least Privilege: By using a service account, you can adhere to the principle of least privilege. This means that you only grant the necessary permissions that the application needs to function. If you update the IAM role on the node, any future applications you deploy in your cluster will also have access to Amazon DynamoDB. This might not be necessary and could pose a security risk.

⦁	Security: If the application is compromised, the attacker only has access to the permissions of the service account that the application uses. If you use the node’s IAM role, an attacker could potentially gain access to all permissions associated with that role.

⦁	Separation of Concerns: Using service accounts allows for a clear separation of concerns. Each application has its own service account and is responsible for its own permissions. This makes it easier to manage and troubleshoot.
# TASK 4.2: USE A SERVICE ACCOUNT TO GRANT PERMISSIONS
Now that you’ve identified the problem, you create a service account that allows your application to add users to the Amazon DynamoDB database.

⦁	To stop streaming the Kubernetes logs, press Ctrl+C.

 Expected output:
None, unless there is an error.

⦁	 Command: Enter the following command to use eksctl to create a service account called dynamo-sa:

eksctl create iamserviceaccount --cluster dev-cluster --namespace default --name dynamo-sa --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/dynamodb-sa-policy --override-existing-serviceaccounts --approve --region ${AWS_REGION}

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

2023-09-05 18:18:47 [ℹ]  1 iamserviceaccount (default/dynamo-sa) was included (based on the include/exclude rules)
2023-09-05 18:18:47 [!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
2023-09-05 18:18:47 [ℹ]  1 task: { 
    2 sequential sub-tasks: { 
        create IAM role for serviceaccount "default/dynamo-sa",
        create serviceaccount "default/dynamo-sa",
    } }2023-09-05 18:18:47 [ℹ]  building iamserviceaccount stack "eksctl-dev-cluster-addon-iamserviceaccount-default-dynamo-sa"
2023-09-05 18:18:47 [ℹ]  deploying stack "eksctl-dev-cluster-addon-iamserviceaccount-default-dynamo-sa"
2023-09-05 18:18:47 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-addon-iamserviceaccount-default-dynamo-sa"
2023-09-05 18:19:17 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-addon-iamserviceaccount-default-dynamo-sa"
2023-09-05 18:19:17 [ℹ]  created serviceaccount "default/dynamo-sa"

# Great work. Now that you’ve created the service account, use the kubectl set command to map it to the backend deployment.

 Learn more: A Kubernetes service account provides an identity for processes that run in a pod. If your pod needs access to AWS services, you can map the service account to an AWS Identity and Access Management identity to grant that access. For more information, see Kubernetes service accounts in the Additional resources section for more information.

⦁	 Command: Enter the following command to configure the backend deployment to use the service account:

kubectl set serviceaccount deployment backend dynamo-sa
 
 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

deployment.apps/backend serviceaccount updated
 
 Note: The kubectl set [RESOURCE/IMAGE/SERVICEACCOUNT] [NAME] command can be used to update existing resources.

⦁	 Command: Enter the following command to display information about the deployment and highlight the line that confirms that the service account has been associated with it:

kubectl describe deployment backend | egrep --color -e '.Service Account.*|$' -e '^'

The following list describes each flag used on the command you just entered:

⦁	kubectl describe deployment backend: Prints verbose description of the backend deployment.

⦁	egrep --color -e: Highlights target string from output.

⦁	‘.Service Account.*|$’ -e ‘^’: A regular expression that searches for a string containing Service Account.

 Expected output:
Explain
******************************
**** This is OUTPUT ONLY. ****
******************************

Name:                   backend
Namespace:              default
CreationTimestamp:      Tue, 05 Sep 2023 15:12:27 +0000
Labels:                 app=backend
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=backend
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app=backend
  Service Account:  dynamo-sa
  Containers:
   backend:
    Image:      mcr.microsoft.com/dotnet/sdk:6.0
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      -c
    Args:
      mkdir /app ; cd /app ; apt update ; apt install wget -y curl unzip ; curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" ; unzip awscliv2.zip ; ./aws/install ; aws s3 cp s3://sourcecodebucket-111111111111/DirectoryBackend.zip . ; unzip DirectoryBackend.zip ; cd DirectoryBackend/ ; dotnet run
    Environment:
      ASPNETCORE_URLS:  http://+:80
    Mounts:             <none>
  Volumes:              <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   backend-6888895ff9 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  12m   deployment-controller  Scaled up replica set backend-6888895ff9 to 1
  Normal  ScalingReplicaSet  12m   deployment-controller  Scaled down replica set backend-65946ddc59 to 1 from 2
  Normal  ScalingReplicaSet  12m   deployment-controller  Scaled up replica set backend-6888895ff9 to 2 from 1
  Normal  ScalingReplicaSet  12m   deployment-controller  Scaled down replica set backend-65946ddc59 to 0 from 1

# Now that you’ve confirmed that the service account has been applied to the backend deployment, let’s confirm that the application is able to add users to the database.

⦁	Once again, return to the browser tab connected to the application and choose the Back button to go back to the Corporate Directory - Add/Edit page.

⦁	Create a user with the following details. This time it should work:

⦁	For Full Name, enter 
Arnav Desai
.
⦁	For Location, enter 
Las Vegas
.
⦁	For Job Title, enter 
DevOps Engineer
.
⦁	Choose the checkbox next to Linux User.

⦁	Scroll to the bottom of the page and select Save.

The web page shows that Arnav Desai has been added to the employee directory. This confirms that your EKS application is up and running.

![Screenshot (21)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/08f7a54f-b14c-4265-9e63-405b0536130c)

*Image description: In the preceding image, the application shows that Arnav Desai has been saved to the employee directory.*
# Congratulations! You were able to troubleshoot the application successfully
# Additional resources
https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html

https://docs.aws.amazon.com/eks/latest/userguide/setting-up.html

https://github.com/eksctl-io/eksctl

https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html

https://docs.aws.amazon.com/eks/latest/userguide/service-accounts.html

https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html

https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html

https://kubectl.docs.kubernetes.io/references/kubectl/apply/

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

https://docs.aws.amazon.com/cloud9/latest/user-guide/security-iam.html
