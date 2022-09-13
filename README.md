# What is that repository?

- We are gone create an EKS cluster on AWS
- Create a repository for yaml files
- Make a deployment with CDK to AWS EKS
- How to create repo structure with cdk8s
# Prerequisites

## INSTALL CDK AND CDK8S

- Set AWS account configs first. Like install aws cli and set aws config files. This doc will not cover it. Like 
- You should have node and npm on the environment
- Run to install --> ``` npm install -g aws-cdk ``` to verify --> ```cdk --version```
- Install cdk8s --> ```npm install -g cdk8s-cli``` to verify --> ```cdk8s --version```
- Run to install pipenv ```pip3 install pipenv```

## How to create this repo from 0

### To cdk8s folder

- Go to your working directory. Mine is docker-k8s-Workdir

- ```mkdir cdk8s``` then ```cdk8s init python-app```
- ```├── Pipfile
     ├── Pipfile.lock
     ├── cdk8s.yaml
     ├── dist
     │   └── cdk8s.k8s.yaml
     ├── help
     ├── imports
     │   └── k8s
     │       ├── __init__.py
     │       ├── _jsii
     │       │   ├── __init__.py
     │       │   └── k8s@0.0.0.jsii.tgz
     │       └── py.typed
     └── main.py```
    ### Below is the sample tree format that is created with init command.

### Creating Deployment Files

- Check the file that is created. 
- Check ```cdk8s/main.py``` We are gone use ```{"app": "cdk8s"}```
 as label. You can change if you want. 
 - ```public.ecr.aws/s9u7u6x1/sample_app_001``` we are gone use this opensource repo for image on yaml files.
 - Change your ```main.py``` with this repos. Doing every line manually is recommended.
 - Run ```cdk8s synth``` then check dist folder
   ```
      .
      ├── cdk8s-deployment.k8s.yaml
      └── cdk8s-service.k8s.yaml 
    ```
    This file is generated with main.py configs that you gave.

### Creating EKS on AWS

- ```mkdir k8s-cluster``` then ```cdk init app --language=python``` to create structure. Then run ```python3 -m venv .venv ```
-  Change your file with ```k8s-cluster/k8s_cluster/k8s_cluster_stack.py``` or related file. It ll differ according to your environment cd.
- Change your file with ```k8s-cluster/app.py```. Replace account id and region with yours.
- Run ```cdk diff``` check the output. Then run ```cdk deploy``` again check the summary and write y when it asked on prompt. (If cdk deploy command gives error, comment in 31-40 lines on this file k8s_cluster_stack.py  )
- Check the output it will recomment some commands.
- Run```aws eks update-kubeconfig --name ClusterStack-cluster --region <region> --role-arn arn:aws:iam::<Account id>:role/ClusterStack-iam``` something like this. You should see added ew context arn:aws...  output from this command. To verify run ```kubectl get all```

# DEPLOYMENT

-  If you k8s_cluster_stack.py file has comment uncomment them.
-  Go ```k8s-cluster``` directory
- Run ```cdk diff``` check output then run ```cdk deploy```
- Check your deployments with ```kubectl get all```

## To RBAC issues

- Run ```aws sts get-caller-identity``` to learn your aws identity details.
- Run ```kubectl edit configmap aws-auth --namespace kube-system```. Update mapUsers section with this:
    ```
    mapUsers: |
      - userarn: arn:aws:iam::<Account-id>:user/<aws-Username>
        username: <aws-Username>
        groups:
          - system:masters
    ```
    Dont forget gaps. Save and exit.
-  Create a policy on aws and add to user or group:
    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "eks:ListFargateProfiles",
                    "eks:DescribeNodegroup",
                    "eks:ListNodegroups",
                    "eks:ListUpdates",
                    "eks:AccessKubernetesApi",
                    "eks:ListAddons",
                    "eks:DescribeCluster",
                    "eks:DescribeAddonVersions",
                    "eks:ListClusters",
                    "eks:ListIdentityProviderConfigs",
                    "iam:ListRoles"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": "ssm:GetParameter",
                "Resource": "arn:aws:ssm:*:<Account-id>:parameter/*"
            }
        ]
    }
    ``` 
    Change id and region
- Run ```kubectl apply -f eks-console-full-access.yaml```
- OR run with updating ```kubectl apply -f eks-console-restricted-access.yaml``` 
- You can follow https://aws.amazon.com/premiumsupport/knowledge-center/eks-api-server-unauthorized-error/ and https://docs.aws.amazon.com/eks/latest/userguide/view-kubernetes-resources.html#view-kubernetes-resources-permissions