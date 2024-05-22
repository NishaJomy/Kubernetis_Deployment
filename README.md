# Kubernetis_Deployment
Create and setup custom VPC with private and public subnet having NAT gateway and Internet gateway.
Deployed and setup MySql database in private subnet using Amazon RDS service.
Create EKS cluster in private subnet and deploy springboot application.
Deployed Internet facing AWS Application load balance to access to application.
Connect to MySql database which deployed in private subnet from local machine.
Here I have created REST API's to add and retrieve ExchangeRate in the MySql database.
erform following steps for application deployment:
Setup Custom VPC in AWS account using VPC service including NAT Gateway and Internet Gateway.

Create a Security Group having inbound rule with access from anywhere. Select VPC created in Step-1 while creating Security group (Note. You can create multiple security groups with restricted access.).

Create a MySql database subnet group having a private subnet. (using Amazon RDS service).

Create a MySql database and use the subnet group created in Step-3. Also use existing security group created in Step-2

Create an EC2 instance key pair and download it on a local machine.

Launch EC2 instance.

Use EC2 instance key pair created in step-5 while launching EC2 instance.
Use the existing security group which was created in Step-2.
Once the EC2 instance and MySql databases are in Running/Active state. Connect MySql database from local machine.

Open terminal on local machine and cd into ec2 key pair directory.
Change permission of ec2 key pair file using command : chmod 0400 ec2-db-key-pair.pem (Note: here ec2-db-key-pair.pem is the file which is downloaded when we created ec2-key. Change name accordingly.)
Then run the following command:
 ssh -i "YOUR_EC2_KEY" -L LOCAL_PORT:RDS_ENDPOINT:REMOTE_PORT EC2_USER@EC2_HOST -N -f 
Replace YOUR_EC2_KEY with your actual key pair name and other values accordingly.
Then open the database client and connect to the database.

Clone this repository.

From the terminal cd into your project directory and build project using command:

 MYSQL_HOSTNAME=127.0.0.1 MYSQL_PORT=3306 MYSQL_DATABASE=ytlecture MYSQL_USERNAME=root MYSQL_PASSWORD=your_password ./gradlew clean build 
or to build without running test run this command ./gradlew clean assemble
Create a docker repository in AWS ECS service. Give it name as springboot-mysql-eks 

Follow the push commands from AWS ECS repository push command options. (make sure docker is started on the local machine.).

AWS Application Load Balancer setup steps:
Tags all the subnet with the following tags:
For Private subnet:
key - kubernetes.io/role/internal-elb
value - 1 
For Public subnet:
kubernetes.io/role/elb
value - 1 
For more and updated information Click here.
Create IAM Policy for AWS Load Balancer Controller. For more information Click here.
Open a new terminal. CD into application repository cluster directory.
Get policy document using command :
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
Or you can use policy from repo using cluster/iam_policy.json (Optional)
Apply the policy using command :
       aws iam create-policy \
      --policy-name AWSLoadBalancerControllerIAMPolicy \
      --policy-document file://iam_policy.json 
Create cluster using command: eksctl create cluster -f cluster.yaml 
Once the cluster is created, then run following commands:
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
    --namespace kube-system \
    --set clusterName=spring-test-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller
kubectl -n kube-system rollout status deployment aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
Once the application load balancer is ready. Open new terminal and cd into project directory.
Run the command to deploy application :
helm install mychart ytchart 
Check the deployments using  kubectl get all
Go into EC2 service in AWS account. Select AWS Load balancer and copy DNS name to test application.
