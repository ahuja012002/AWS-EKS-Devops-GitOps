# AWS-EKS-Devops-GitOps
Detailed description of Traditional CI/CD pipeline and new GitOps for Container based deployments on AWS EKS

## Traditional Devops :

A standard Kubernetes DevOps pipeline involves the following steps:

Step 1: Code Commit – a developer commits code in the repository (i.e., GitHub, AWS CodeCommit). The code contains a Dockerfile, specifying the container image, and a manifest file specifying a deployment object. 

Step 2: Build – a CI/CD tool processes the Dockerfile to create a new version of the container image.

Step 3: Generate artifacts – the CI/CD tool pushes the container image to an image repository. (Amazon ECR)

Step 4: Deploy – The tool deploys manifest files to the Kubernetes cluster (i.e., an Amazon EKS cluster).

The first three steps are part of the continuous integration (CI), while the fourth step is continuous delivery or deployment (CD).

Below is the architecture diagram for standard CI/CD Devops pipeline for EKS Cluster.


![diagram-export-3-28-2025-1_10_32-PM](https://github.com/user-attachments/assets/44d1a791-be29-478f-877a-d622179c0389)


In this Example we will use the below artifacts :

1. Github as the source code repository : For this example we will use the below github repository which contains the code, dockerFile and buildspec.yml file. We will discuss these in detail :
   https://github.com/ahuja012002/AWS-Springboot-3tierApp
2. We will use Amazon ECR as the container image repository.
3. We will use Code pipeline to trigger the build (Code build will be used for building the code and generating the image)
4. We will create buildspec.yml file which contains all the instructions to build and deploy the code on EKS Cluster.
5. We will also create EKS Cluster and corresponding IAM role and RBAC permissions to access the EKS Cluster.

## Creating ECR Repository :

Lets now login to AWS Console to go to ECR. Click Create Repository 


<img width="1710" alt="Screenshot 2025-01-23 at 9 15 54 PM" src="https://github.com/user-attachments/assets/f1ba680b-85a9-4fce-a5a8-c060d7f9d031" />



Give any name to the repository and Click Create. This should create a repository.

<img width="1490" alt="Screenshot 2025-03-28 at 2 02 08 PM" src="https://github.com/user-attachments/assets/b0e957d8-9d11-4dcb-a209-2cbfd7f76dac" />


## Create EKS Cluster 

create EKS Cluster using the below command :
kubectl create cluster <Cluster-name>

## Create Code Pipeline

Navigate to your AWS Console and search for Code Pipeline and Click Create Pipeline and Chose Custom Pipeline

<img width="1151" alt="Screenshot 2025-03-28 at 2 09 11 PM" src="https://github.com/user-attachments/assets/5ba20a31-4701-4738-a7c9-62d0784ddd24" />

Enter any pipeline name and new Service role :

<img width="1264" alt="Screenshot 2025-03-28 at 2 10 23 PM" src="https://github.com/user-attachments/assets/e260643a-c49d-4c10-a776-9bfb61c92fc5" />

For Source Provider, Select Github connected app, Click on Connect app and login with your Github ID and select relevant repository and branch :

<img width="851" alt="Screenshot 2025-03-28 at 2 11 40 PM" src="https://github.com/user-attachments/assets/1c410cf6-9deb-4e0e-9716-e7fd36ca1426" />

For Build stage, Select Other build providers and Select Code build

<img width="841" alt="Screenshot 2025-03-28 at 2 13 35 PM" src="https://github.com/user-attachments/assets/70c86c84-be12-48f0-b280-9e3e404bb7b9" />

Click Create Project and control will go to Code Build. Enter any build project and select new Service Role for Codebuild and enter name.

<img width="945" alt="Screenshot 2025-03-28 at 2 17 28 PM" src="https://github.com/user-attachments/assets/8e140107-1fa1-4bd8-b39c-6a435ff12aab" />


For Build specifications Select buildspec.yml file. This file is present in our code and we will discuss about this.

<img width="888" alt="Screenshot 2025-03-28 at 2 21 15 PM" src="https://github.com/user-attachments/assets/31468bdf-70b1-4f5a-8b5f-303cc5a4ec87" />


We also need to add environment variables which will be referred in our buildspec.yml file :

<img width="733" alt="Screenshot 2025-03-28 at 2 24 10 PM" src="https://github.com/user-attachments/assets/8640dd0a-53e5-40c6-86ab-2694bed7c245" />


One more setting that we need to select is to enable privileged mode to build docker images.

<img width="640" alt="Screenshot 2025-03-28 at 2 24 48 PM" src="https://github.com/user-attachments/assets/02e9d142-daae-419f-8f03-b5f04dc06b3d" />


Then Click on Continue to Code pipeline. There skip everything else and Click Create.

## IAM Roles and RBAC permissions for Kubectl to access EKS Cluster.

Navigate to IAM in AWS Console and Click Create Role. Select AWS Account and This account and Click next.

<img width="1506" alt="Screenshot 2025-03-28 at 2 33 33 PM" src="https://github.com/user-attachments/assets/36e7eb97-bbf7-4618-8e10-bb80b888777e" />

Enter Role name and Click next and Create.

Search for the newly created role and Add permissions and add the below permissions to the role :

<img width="1445" alt="Screenshot 2025-03-28 at 2 37 00 PM" src="https://github.com/user-attachments/assets/ce839aa8-3168-40bc-9693-8be7763de988" />

For the above permissions, we need to create policy and then attach policy to the role.

Next Step is to add the newly created IAM role to the AWS Auth configmap of the EKS Cluster using the below command

eksctl create iamidentitymapping --cluster my-devops-cluster --arn arn:aws:iam::215472211497:role/CodeBuildEKSRole --group system-masters --username build

Now, Lets go to Service role used by Code build. Attach an inline policy 

{
    "Version": "2012-10-17",
    "Statement": [
      {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::XXXXXXX:role/CodeBuildEKSRole"
        }
    ]
}

Also, add the permissions AmazonEC2ContainerRegistryFullAccess to the role. As shown below :

<img width="1510" alt="Screenshot 2025-03-28 at 2 44 22 PM" src="https://github.com/user-attachments/assets/60b8e143-de4d-4194-bd8a-69bea3d38214" />

This should now complete all the required permissions.

## Lets now look at the buildspec.yml file :

<img width="1260" alt="Screenshot 2025-03-28 at 2 47 19 PM" src="https://github.com/user-attachments/assets/8d45bb30-6715-410c-9471-5841195a988e" />


In the install phase, we are installing the required softwares .
In the pre-build phase we are logging into Amazon ECR
In the build phase, we are building the docker image and tagging with the latest timestamp. Also, we are replacing the manifest file with the latest image URI.
In the post build phase, we are pushing the docker image to Amazon ECR
Finally, we are assuming the Kubectl role, updating kube config with relevant cluster details and deploying manifest file

This completes our CI/CD Devops Pipeline. To test we can change some file in github and test.

Open CodeBuild and Tail logs and it show success and manifest file deployed in EKS cluster.

<img width="1383" alt="Screenshot 2025-03-28 at 4 07 34 PM" src="https://github.com/user-attachments/assets/cf978034-6412-4909-a431-80dfacb771b2" />

We can check the status of pods deployed using

Kubectl get pods

Also, we can check load balancer service url using

kubectl get svc

We can now access our application

![image](https://github.com/user-attachments/assets/176d04e4-30e5-46a4-bdb8-1eaa8cb86217)


### This successfully completes our CI/CD Devops pipeline

## GitOps

GitOps enables development and operations teams to use Git—an open source version control and distributed code management system—for declarative configuration driving application delivery and cluster management.

Git tracks all of the changes to the application and environment configuration via commits and pull requests which triggers a synchronize event. A GitOps workflow enables teams to manage software deployment and infrastructure provisioning in an automated and reliable manner.

Technically, GitOps requires a Git repository, an environment supporting declarative configuration (like Kubernetes), and a GitOps agent that synchronizes between the Git repository, which is the source of truth, and the current state of the cluster.

### ArgoCD for Gitops

Argo CD automates the process of detecting changes to resources and reapplying the desired state when drift detection is enabled. Argo CD works with Argo Rollouts, a progressive delivery tool that can handle blue/green and canary deployments out of the box, and Argo Workflows, a workflow engine that orchestrates parallel jobs in Kubernetes.

We will now implement GitOps pipeline with Github, AWS Code pipeline and Code Build, Argo CD. Below is the architecture diagram to implement Gitops

![image](https://github.com/user-attachments/assets/08c6170f-a03f-4729-9635-5714257d1849)



CI part of CI/CD will remain same in Gitops as Traditional Devops. The difference comes in CD part where instead of Codebuild pushing the manifest file in EKS cluster, ArgoCD pulls the latest changes from github repo and deploys on the EKS cluster.

## Installation of ArgoCD on EKS Cluster

Lets create EKS Cluster as shown below :

<img width="323" alt="Screenshot 2025-03-24 at 3 38 56 PM" src="https://github.com/user-attachments/assets/fb5ebc28-4484-4291-ab9e-8beb3a7fada0" />


Lets install ArgoCD on the EKS Cluster as shown below . We will follow official argoCd installation instructions as shown in the link https://argo-cd.readthedocs.io/en/stable/getting_started/

<img width="1091" alt="Screenshot 2025-03-24 at 3 38 47 PM" src="https://github.com/user-attachments/assets/3d54c68c-1b9d-4ee8-8e10-36cff12db684" />

Once, ArgoCD is installed , We need to setup ArgoCD CLI as per instructions in the above link and generate password :

Once password is generated, we can login into the argoCD

<img width="1692" alt="Screenshot 2025-03-24 at 3 31 59 PM" src="https://github.com/user-attachments/assets/17ad27d2-7a03-49b7-b77d-6b4d389c4a1c" />


## Lets now look at the steps needed in AWS Code pipeline and AWS Code build

As discussed, most of the steps would remain same as Devops. In Code pipeline, we need to create webhook triggers as shown in the below screenshot :

<img width="762" alt="Screenshot 2025-03-29 at 5 23 33 PM" src="https://github.com/user-attachments/assets/6fdca2b6-c07f-42ce-9dad-39d5bbfde037" />

This shows that dont start the pipeline for the below conditions. We need to enable this filter so that it should not go into infinite loop.
Any commits in manifest yaml file should not trigger the pipeline as this file will be committed during the build process.

## Review builspec-gitops.yaml

<img width="1376" alt="Screenshot 2025-03-29 at 5 25 58 PM" src="https://github.com/user-attachments/assets/b35a0176-ea50-450b-a04a-9dc3b79014c6" />

As discussed, most of it is same as Traditional Devops buildspec.yml file. However, in this we are not pushing the updated manifest file and not using kubectl to deploy on EKS cluster.
Instead, we are commiting the updated file and pushing it to github.

### Setup ArgoCD.

Once all this setup is completed, we can login into ArgoCd and create an application.

![image](https://github.com/user-attachments/assets/9e785713-fe03-49bf-9173-647a685e6219)

We can enter the name of the application. This can be any name and we can also use the name defined in manifest file.

Source should be our github repo url. Branch can be master or main depending on where our codebase is present.

path should be the path where manifest file is present. if it is root path, we can enter as ./

Cluster name should be default cluster url and namespace is default.

Once all these values are added, we can click create. This should start deploying our application on EKS cluster as shown below :

<img width="1589" alt="Screenshot 2025-03-24 at 3 37 43 PM" src="https://github.com/user-attachments/assets/ec4453d3-5f0a-41d2-8a52-432e811c701c" />


<img width="1515" alt="Screenshot 2025-03-24 at 3 37 36 PM" src="https://github.com/user-attachments/assets/e0a497c8-f4cd-4968-ba86-11ac76638096" />

We can now test our application and also can change some file on github and can see if it relects on the page.

<img width="1519" alt="Screenshot 2025-03-24 at 3 37 29 PM" src="https://github.com/user-attachments/assets/bf921349-c768-4835-9a7f-3a4c5583214a" />

### Congratulations, We have now completed gitops pipeline for our EKS Cluster.

So, We see that Gitops is not to replace Devops but it makes Devops better by  enhancing it by incorporating Git throughout the software delivery process, making it easier to orchestrate projects and keep them in sync. The end goal is to achieve smoother, faster, and more reliable software development and delivery.






