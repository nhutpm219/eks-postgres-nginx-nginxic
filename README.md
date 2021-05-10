# Learn Terraform - Provision an EKS Cluster

This repo is a companion repo to the [Provision an EKS Cluster learn guide](https://learn.hashicorp.com/terraform/kubernetes/provision-eks-cluster), containing
Terraform configuration files to provision an EKS cluster on AWS.

![image](https://user-images.githubusercontent.com/70093183/117658251-b8f98a00-b1c4-11eb-9f3e-15332ec92cea.png)

1. Install EKS
- Terraform plan
- Terraform apply

2. Configure kubectl
- Run the following command to retrieve the access credentials for your cluster and automatically configure kubectl.
$ aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)

3. Deploy and access Kubernetes Dashboard
- Deploy Kubernetes Metrics Server
    The Kubernetes Metrics Server, used to gather metrics such as cluster CPU and memory usage
    over time, is not deployed by default in EKS clusters.
$ wget -O v0.3.6.tar.gz https://codeload.github.com/kubernetes-sigs/metrics-server/tar.gz/v0.3.6 && tar -xzf v0.3.6.tar.gz
$ kubectl apply -f metrics-server-0.3.6/deploy/1.8+/
    Verify that the metrics server has been deployed. If successful, you should see something like this.
$ kubectl get deployment metrics-server -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           4s

- Deploy Kubernetes Dashboard
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml

- Authenticate the dashboard
    Create a ClusterRoleBinding
$ kubectl apply -f https://raw.githubusercontent.com/hashicorp/learn-terraform-provision-eks-cluster/master/kubernetes-dashboard-admin.rbac.yaml
    Generate the authorization token
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep service-controller-token | awk '{print $1}')
    Create a proxy server that will allow you to navigate to the dashboard from the browser 
$ kubectl proxy
    You should be able to access the Kubernetes dashboard here 
(http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/).
    Select "Token" on the Dashboard UI then copy and paste the entire token you receive into the dashboard authentication screen to sign in.

3. Create a secret
$ kubectl create secret generic postgresql-secrets --from-literal=password=admin123

4. Install StorageClass (can ignore for this lab)
$ kubectl apply -f storageclass.yaml

5. Install statefulset and headless postgresql service
$ kubectl apply -f postgres-headless-statefulset.yaml

6. Install 2 nginx server link to postgresql database
$ kubectl apply -f postgrems-nginx.yaml
$ kubectl apply -f postgres-nginx02.yaml

7. Create Nginx Ingress controller (modified use-proxy-protocol: 'false')
$ kubectl apply -f nginx-ingress-controller.yaml

8. Create ingress
$ kubectl apply -f ingress.yaml




1.1. Terraform state list
data.aws_availability_zones.available
data.aws_eks_cluster.cluster
data.aws_eks_cluster_auth.cluster
aws_security_group.all_worker_mgmt
aws_security_group.worker_group_mgmt_one
aws_security_group.worker_group_mgmt_two
random_string.suffix
module.eks.data.aws_ami.eks_worker
module.eks.data.aws_ami.eks_worker_windows
module.eks.data.aws_caller_identity.current
module.eks.data.aws_iam_policy_document.cluster_assume_role_policy
module.eks.data.aws_iam_policy_document.cluster_elb_sl_role_creation[0]
module.eks.data.aws_iam_policy_document.workers_assume_role_policy
module.eks.data.aws_partition.current
module.eks.data.template_file.userdata[0]
module.eks.data.template_file.userdata[1]
module.eks.aws_autoscaling_group.workers[0]
module.eks.aws_autoscaling_group.workers[1]
module.eks.aws_eks_cluster.this[0]
module.eks.aws_iam_instance_profile.workers[0]
module.eks.aws_iam_instance_profile.workers[1]
module.eks.aws_iam_policy.cluster_elb_sl_role_creation[0]
module.eks.aws_iam_role.cluster[0]
module.eks.aws_iam_role.workers[0]
module.eks.aws_iam_role_policy_attachment.cluster_AmazonEKSClusterPolicy[0]
module.eks.aws_iam_role_policy_attachment.cluster_AmazonEKSServicePolicy[0]
module.eks.aws_iam_role_policy_attachment.cluster_AmazonEKSVPCResourceControllerPolicy[0]
module.eks.aws_iam_role_policy_attachment.cluster_elb_sl_role_creation[0]
module.eks.aws_iam_role_policy_attachment.workers_AmazonEC2ContainerRegistryReadOnly[0]
module.eks.aws_iam_role_policy_attachment.workers_AmazonEKSWorkerNodePolicy[0]
module.eks.aws_iam_role_policy_attachment.workers_AmazonEKS_CNI_Policy[0]
module.eks.aws_launch_configuration.workers[0]
module.eks.aws_launch_configuration.workers[1]
module.eks.aws_security_group.cluster[0]
module.eks.aws_security_group.workers[0]
module.eks.aws_security_group_rule.cluster_egress_internet[0]
module.eks.aws_security_group_rule.cluster_https_worker_ingress[0]
module.eks.aws_security_group_rule.workers_egress_internet[0]
module.eks.aws_security_group_rule.workers_ingress_cluster[0]
module.eks.aws_security_group_rule.workers_ingress_cluster_https[0]
module.eks.aws_security_group_rule.workers_ingress_self[0]
module.eks.kubernetes_config_map.aws_auth[0]
module.eks.local_file.kubeconfig[0]
module.eks.null_resource.wait_for_cluster[0]
module.eks.random_pet.workers[0]
module.eks.random_pet.workers[1]
module.vpc.aws_eip.nat[0]
module.vpc.aws_internet_gateway.this[0]
module.vpc.aws_nat_gateway.this[0]
module.vpc.aws_route.private_nat_gateway[0]
module.vpc.aws_route.public_internet_gateway[0]
module.vpc.aws_route_table.private[0]
module.vpc.aws_route_table.public[0]
module.vpc.aws_route_table_association.private[0]
module.vpc.aws_route_table_association.private[1]
module.vpc.aws_route_table_association.private[2]
module.vpc.aws_route_table_association.public[0]
module.vpc.aws_route_table_association.public[1]
module.vpc.aws_route_table_association.public[2]
module.vpc.aws_subnet.private[0]
module.vpc.aws_subnet.private[1]
module.vpc.aws_subnet.private[2]
module.vpc.aws_subnet.public[0]
module.vpc.aws_subnet.public[1]
module.vpc.aws_subnet.public[2]
module.vpc.aws_vpc.this[0]

