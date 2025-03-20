# Ingress controller

REGION_CODE=us-east-1
CLUSTER_NAME=expense
ACC_ID=688567281358


NOTE: ALB and EKS CLUSTER are both are 2 different services we need to follow below:

1. OIDC Provider   (it was type of Authentication service which will connect AWS and cluster)

eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve

2. IAM Policy

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.12.0/docs/install/iam_policy.json

3. Create IAM role

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

4. Provide access to EKS through IAM policy 

eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve

### Install Drivers

1.Add the EKS chart repo to Helm:

helm repo add eks https://aws.github.io/eks-charts

2. Install AWS Load Balancer Controller

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller