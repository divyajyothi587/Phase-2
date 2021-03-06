* Referecne: https://medium.com/containermind/how-to-create-a-kubernetes-cluster-on-aws-in-few-minutes-89dda10354f4

* Check the logs if you have any issues: https://github.com/DevOpsBasicSetup/Phase-2/blob/master/kubernetes/KOPS/kops-logs.txt

#### Step-1: Create the Access Key Id & Secret Key in AWS console. 

	Create a user and group with below permissions:
	
	AmazonEC2FullAccess
	AmazonRoute53FullAccess
	AmazonS3FullAccess
	IAMFullAccess
	AmazonVPCFullAccess

	AWS Access Key ID: AKIA*******WCOYQ

	AWS Secret Access Key: yWS5QBlC******GSs781
	
#### Step-2: Launch an Ubuntu instance with type t2.medium. And connect to the machine.

![image](https://user-images.githubusercontent.com/24622526/45141750-4a0f3c00-b1a6-11e8-8a4f-91aeefb25bc6.png)

#### Step-3: Install awscli, kops, Kubelet, Kubectl and Kubernetes-cni.

	apt-get update

	apt-get install -y apt-transport-https

	curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add

	vi /etc/apt/sources.list.d/kubernetes.list

	deb http://apt.kubernetes.io/ kubernetes-xenial main

	apt-get update

	apt-get install -y kubelet kubectl kubernetes-cni
	
	apt install awscli -y
	
	curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
	chmod +x kops-linux-amd64
	sudo mv kops-linux-amd64 /usr/local/bin/kops
	
	kubectl version
	
	kops version

Step-4: Configure the AWS credentials: Run the command and enter the access key id & secret key: aws configure

	Output:
	root@ip-172-31-16-220:~# aws configure
	AWS Access Key ID [None]: *******************
	AWS Secret Access Key [None]: yWS5**************************8GSs781
	Default region name [None]: us-west-1
	Default output format [None]: json

#### Step-5: Create an AWS S3 bucket for kops to persist its state:

	5.1. Create environment variable: export bucket_name=svn-kops-state-store
		
	5.2. Create s3 bucket: aws s3api create-bucket --bucket ${bucket_name} --region us-west-1
	
	     Note: Region wise there are some contraints/restricts on s3 bucket creation. ex: aws s3api create-bucket --bucket ${bucket_name} --region us-east-2 --create-bucket-configuration LocationConstraint=us-east-2

	5.3. Enable versioning for the above S3 bucket: aws s3api put-bucket-versioning --bucket ${bucket_name} --versioning-configuration Status=Enabled

#### Step-6: Create environment variables for bucket name & cluster name. You can update the below variables in vi ~/.profile Or vi ~/.bash_profile

	export bucket_name=svn-kops-state-store
	export KOPS_CLUSTER_NAME=venkat.k8s.local	
	export KOPS_STATE_STORE=s3://${bucket_name}

#### Step-7: ssh-keygen -t rsa

#### Step-8: Creating k8s cluster in AWS: 

	8.1. kops create cluster --node-count=3 --node-size=t2.medium --zones=us-west-1a --name=${KOPS_CLUSTER_NAME}

	8.2. Finally configure your cluster with: kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
	
	8.3. See the cluser info: kops edit cluster --name ${KOPS_CLUSTER_NAME}

	8.4. kubectl get nodes (wait for sometime to create the cluster the and the cluster available.)
	
	8.5. Above command mentioned at the point 8.2. may take some time to create the required infrastructure resources on AWS. Execute the validate command to check its status and wait until the cluster becomes ready: kops validate cluster
	
	8.6. kubectl cluster-info
	
![image](https://user-images.githubusercontent.com/24622526/45141975-dae61780-b1a6-11e8-8733-7f0badb52c2a.png)

![image](https://user-images.githubusercontent.com/24622526/45142130-40d29f00-b1a7-11e8-89a5-eb9da3b859e6.png)

#### Step-9: Create deployment:

	kubectl create deployment nginx --image=nginx
	
	kubectl get deploy
	
#### Step-10: Create Service:

	kubectl create service nodeport nginx --tcp=80:80
	
	kubectl get svc
	
	curl http://[publicIP]:[nodePort]  //public IP of any host which is in the cluster.
	
	Launch the url in any browser to access your app: curl http://[publicIP]:[nodePort]
	
![image](https://user-images.githubusercontent.com/24622526/45142066-1254c400-b1a7-11e8-8a17-1b482cdfed30.png)

	
#### Step-11: Delete the cluster:

	kops delete cluster --name=${KOPS_CLUSTER_NAME} --state=${KOPS_STATE_STORE} --yes
	
![image](https://user-images.githubusercontent.com/24622526/45142204-737c9780-b1a7-11e8-9041-2fdcccdaa19c.png)
