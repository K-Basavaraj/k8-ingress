# Ingress Controller 

```
Create an IAM OIDC provider. You can skip this step if you already have one for your cluster
 eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster expense-1 \
    --approve

Here: Ingress controller is outside of the EKS to get access we are using the oidc provider
```

```
Download an IAM policy for the LBC using one of the following commands:
If your cluster is in any other region:
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.13.2/docs/install/iam_policy.json
```
```
Create an IAM policy named AWSLoadBalancerControllerIAMPolicy. If you downloaded a different policy, replace iam-policy with the name of the policy that you downloaded.

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

```
Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step.

eksctl create iamserviceaccount \
--cluster=expense-1 \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::<accont_id>:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

```
Add the EKS chart repo to Helm
helm repo add eks https://aws.github.io/eks-charts
```

```
Helm install command for clusters with IRSA:
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=expense-1 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```
```
output: 
NAME: aws-load-balancer-controller
LAST DEPLOYED: Fri May 30 12:18:06 2025
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWS Load Balancer controller installed!

example: kubectl get pods -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
aws-load-balancer-controller-xxxxc-yyq   1/1     Running   0          3m41s
aws-load-balancer-controller-xxxxxc-yyg   1/1     Running   0          3m41s
these are the responsible to to connect between eks cluser and your load balancer.
```