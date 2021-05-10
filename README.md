# Learn Terraform - Provision an EKS Cluster

This repo is a companion repo to the [Provision an EKS Cluster learn guide](https://learn.hashicorp.com/terraform/kubernetes/provision-eks-cluster), containing
Terraform configuration files to provision an EKS cluster on AWS.

![image](https://user-images.githubusercontent.com/70093183/117658251-b8f98a00-b1c4-11eb-9f3e-15332ec92cea.png)

1. Install EKS
- Terraform plan
- Terraform apply
- 
nhutpm@nhutpm:~/terraform-practice02$ kubectl get svc

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE

kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   35m


2. Configure kubectl

- Run the following command to retrieve the access credentials for your cluster and automatically configure kubectl.
- 
$ aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)

3. Deploy and access Kubernetes Dashboard

- Deploy Kubernetes Metrics Server

    + The Kubernetes Metrics Server, used to gather metrics such as cluster CPU and memory usage
      over time, is not deployed by default in EKS clusters.
      
$ wget -O v0.3.6.tar.gz https://codeload.github.com/kubernetes-sigs/metrics-server/tar.gz/v0.3.6 && tar -xzf v0.3.6.tar.gz

$ kubectl apply -f metrics-server-0.3.6/deploy/1.8+/

nhutpm@nhutpm:~/terraform-practice02$ kubectl apply -f metrics-server-0.3.6/deploy/1.8+/
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created


    + Verify that the metrics server has been deployed. If successful, you should see something like this.
$ kubectl get deployment metrics-server -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           4s

- Deploy Kubernetes Dashboard
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created

- Authenticate the dashboard
    + Create a ClusterRoleBinding
$ kubectl apply -f https://raw.githubusercontent.com/hashicorp/learn-terraform-provision-eks-cluster/master/kubernetes-dashboard-admin.rbac.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created

    + Generate the authorization token
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep service-controller-token | awk '{print $1}')
Name:         service-controller-token-fblsn
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=service-controller
              kubernetes.io/service-account.uid=39acc230-c784-40db-b4fe-a21df013181c

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlkxUWgtU3E2OHRmcXJQQUJISGkxd3ctY3E5VWJNT1dXRUVmMVloSVZFUkkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJzZXJ2aWNlLWNvbnRyb2xsZXItdG9rZW4tZmJsc24iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoic2VydmljZS1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMzlhY2MyMzAtYzc4NC00MGRiLWI0ZmUtYTIxZGYwMTMxODFjIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOnNlcnZpY2UtY29udHJvbGxlciJ9.Pb8umflrsYfYYv1_XX3VU9FMMcQtepb6H--ooX_YranEw7W_Ave6Bxb_bTeL0M7gSe-em61Go8zjco0M7g6zADXHKrWTN747NeUVCKrk-D5VxYWkBgyfWSn3n4xNT2QyjOVhDaLJPYe914iTKBGl5uMs2KKP_OtPwBprWKK5d5jUtpQXMntqqfC2wFtV_m2XMN4fVXv7XLapdqIdN4PbxGppZqIljlaH4XAxZpmi0CAu9RspY1PsUWapkVw4id4X7AjonVv2-cO2cuAtennA8iyQrK90qBygwjaUdUXCja5mBZNvmIljX177wwGWJcU0e9mUj28PNZlYgedFo0z6Ow


    + Create a proxy server that will allow you to navigate to the dashboard from the browser 
$ kubectl proxy
    You should be able to access the Kubernetes dashboard here 
(http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/).
    Select "Token" on the Dashboard UI then copy and paste the entire token you receive into the dashboard authentication screen to sign in.
![image](https://user-images.githubusercontent.com/70093183/117658513-037b0680-b1c5-11eb-88df-90e03cdc263c.png)

![image](https://user-images.githubusercontent.com/70093183/117658540-0a097e00-b1c5-11eb-87c3-0b032db5580a.png)
    

3. Create a secret
$ kubectl create secret generic postgresql-secrets --from-literal=password=admin123
secret/postgresql-secrets created

4. Install StorageClass (can ignore for this lab)
$ kubectl apply -f storageclass.yaml
storageclass.storage.k8s.io/ssd created
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  45m
ssd             kubernetes.io/aws-ebs   Delete          Immediate              false                  20s

5. Install statefulset and headless postgresql service
$ kubectl apply -f postgres-headless-statefulset.yaml
service/postgresql-nginx created
statefulset.apps/postgresql-nginx created

6. Install 2 nginx server link to postgresql database
$ kubectl apply -f postgrems-nginx.yaml
$ kubectl apply -f postgres-nginx02.yaml
nhutpm@nhutpm:~/terraform-practice02$ kubectl get svc
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes         ClusterIP   172.20.0.1       <none>        443/TCP    55m
nginx01            ClusterIP   172.20.199.180   <none>        80/TCP     26s
nginx02            ClusterIP   172.20.101.116   <none>        80/TCP     7s
postgresql-nginx   ClusterIP   None             <none>        5432/TCP   9m10s

7. Create Nginx Ingress controller (modified use-proxy-protocol: 'false')
$ kubectl apply -f nginx-ingress-controller.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created

nhutpm@nhutpm:~/terraform-practice02$ kubectl get svc --namespace=ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP                                                                    PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   172.20.29.177   a17b73dceb16d44ef8b6050df3fb7c45-1588267959.ap-southeast-1.elb.amazonaws.com   80:30033/TCP,443:31572/TCP   3m46s
ingress-nginx-controller-admission   ClusterIP      172.20.178.10   <none>                                                                         443/TCP                      3m46s

8. Create ingress
$ kubectl apply -f ingress.yaml
ingress.networking.k8s.io/nginx-ingress created

nhutpm@nhutpm:~/terraform-practice02$ kubectl describe ingress
Name:             test-ingress-3
Namespace:        default
Address:          a17b73dceb16d44ef8b6050df3fb7c45-1588267959.ap-southeast-1.elb.amazonaws.com
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host  Path  Backends
  ----  ----  --------
  *     
        /nginx01   nginx01:80 (<none>)
        /nginx02   nginx02:80 (<none>)
        /(.*)      nginx01:80 (<none>)
Annotations:
  nginx.ingress.kubernetes.io/use-regex:             true
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{"kubernetes.io/ingress.class":"nginx","nginx.ingress.kubernetes.io/rewrite-target":"/$1","nginx.ingress.kubernetes.io/ssl-redirect":"false","nginx.ingress.kubernetes.io/use-regex":"true"},"name":"test-ingress-3","namespace":"default"},"spec":{"rules":[{"http":{"paths":[{"backend":{"serviceName":"nginx01","servicePort":80},"path":"/nginx01","pathType":"Prefix"},{"backend":{"serviceName":"nginx02","servicePort":80},"path":"/nginx02","pathType":"Prefix"},{"backend":{"serviceName":"nginx01","servicePort":80},"path":"/(.*)","pathType":"Prefix"}]}}]}}

  kubernetes.io/ingress.class:                 nginx
  nginx.ingress.kubernetes.io/rewrite-target:  /$1
  nginx.ingress.kubernetes.io/ssl-redirect:    false
Events:                                        <none>


nhutpm@nhutpm:~/terraform-practice02$ kubectl get all
NAME                           READY     STATUS      RESTARTS   AGE
pod/busybox                    0/1       Completed   0          9h
pod/nginx01-7d9d4bdd5f-dmlcr   1/1       Running     0          8h
pod/nginx01-7d9d4bdd5f-jjcqk   1/1       Running     0          8h
pod/nginx02-76c8fccccc-cg8xt   1/1       Running     0          10h

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kubernetes         ClusterIP   172.20.0.1       <none>        443/TCP    11h
service/nginx01            ClusterIP   172.20.199.180   <none>        80/TCP     10h
service/nginx02            ClusterIP   172.20.101.116   <none>        80/TCP     10h
service/postgresql-nginx   ClusterIP   None             <none>        5432/TCP   10h

NAME                      READY     UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx01   2/2       2            2           10h
deployment.apps/nginx02   1/1       1            1           10h

NAME                                 DESIRED   CURRENT   READY     AGE
replicaset.apps/nginx01-6888d59d55   0         0         0         10h
replicaset.apps/nginx01-7d9d4bdd5f   2         2         2         8h
replicaset.apps/nginx02-76c8fccccc   1         1         1         10h

NAME                                READY     AGE
statefulset.apps/postgresql-nginx   0/2       10h

9. Access to ExternalIP
![image](https://user-images.githubusercontent.com/70093183/117660199-11ca2200-b1c7-11eb-8e4a-a404e02a92dd.png)




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

