## trying out aws logger

Create an eks cluster
`eksctl create cluster -f eks-cluster-config.yaml`

Create namespace and apply configmap
`kubectl apply -f fluentbit-config.yaml`

Check configmap
`kubectl -n aws-observability get cm`

Create permission
cd ~/Documents/aws_logger/
`curl -o permissions.json \
     https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/cloudwatchlogs/permissions.json`

aws iam create-policy \
        --policy-name FluentBitEKSFargate \
        --policy-document file://permissions.json 

aws iam attach-role-policy \
        --policy-arn arn:aws:iam::123456789012:policy/FluentBitEKSFargate \
        --role-name eksctl-fluentbit-cluster-FargatePodExecutionRole-<Account_id>

Create deployment and expose it
`kubectl -n demo apply -f logger-server.yaml && kubectl -n demo expose deploy logger-server`

Check pod
`kubectl -n demo get po -l app=nginx`
`kubectl -n demo describe <pod id>`


Test using 3 commands/terminal
```
# [terminal 1] forward the logger-server traffic locally:
$ kubectl -n demo port-forward svc/logger-server 8080:80 
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
```
```
# [terminal 2] watch logs locally:
$ kubectl -n demo logs deploy/logger-server -f
127.0.0.1 - - [25/Nov/2020:16:03:41 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.64.1" "-"
127.0.0.1 - - [25/Nov/2020:16:03:42 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.64.1" "-"
127.0.0.1 - - [25/Nov/2020:16:03:43 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.64.1" "-"
```
```
# [terminal 3] generate HTTP traffic:
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```