Create a task role that allows the task to write traces to AWS X-Ray.  Replace *<role_name>* with your role name. 

```
export TASK_ROLE_NAME=$(aws iam create-role --role-name <role_name> --assume-role-policy-document file://ecs-trust-pol.json | jq -r '.Role.RoleName')
export XRAY_POLICY_ARN=$(aws iam create-policy --policy-name --policy-document file://xray-pol.json | jq -r '.Policy.Arn')
aws iam attach-role-policy --role-name $TASK_ROLE_NAME --policy-arn $XRAY_POLICY_ARN
```

If this is your first time using ECS you will need to create the ECS Task Execution Role

```
aws iam create-role --role-name ecsTaskExecutionRole --assume-role-policy-document file://ecs-trust-pol.json
export ECS_EXECUTION_POLICY_ARN=$(aws iam list-policies --scope AWS --query 'Policies[?PolicyName==`AmazonECSTaskEtionRolePolicy`].Arn' | jq -r '.[]')
aws iam attach-role-policy --role-name ecsTaskExecutionRole --policy-arn $ECS_EXECUTION_POLICY_ARN
```

Get a list of subnets in a VPC.  Replace *<vpc_id>* with the vpc id of the vpc where you intend to deploy the services.

```
aws ec2 describe-subnets --query 'Subnets[?VpcId==`<vpc_id>`].SubnetId'
```


aws ec2 describe-security-groups --filters Name=vpc-id,Values=vpc-7bc1da1d --query 'SecurityGroups[*].GroupId'
aws ec2 describe-security-groups --filter Name=group-id,Values=sg-09fa0c77 --query 'SecurityGroups[*].IpPermissions'
#verify sg has the appropriate inbound rules

#create alb
aws elbv2 create-load-balancer --name service-b-lb --subnets subnet-f8bc52a2 subnet-7c480740 --security-groups sg-09fa0c77 \
--scheme internet-facing --type application
#create target group
aws elbv2 create-target-group --name service-b-tg --protocol HTTP --port 8080 --vpc-id vpc-7bc1da1d
#create listener
aws elbv2 create-listener --load-balancer-arn \
arn:aws:elasticloadbalancing:us-east-1:820537372947:loadbalancer/app/service-b-lb/5c7afe2e4c2d4f77 --protocol HTTP \
--port 80 --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:820537372947:targetgroup/service-b-tg/3a30e651d497781a

#launch service-b
ecs-cli compose service up --deployment-max-percent 100 --deployment-min-healthy-percent 0 --load-balancer-name service-b-lb \
--target-group-arn arn:aws:elasticloadbalancing:us-east-1:820537372947:targetgroup/service-b-tg/3a30e651d497781a --launch-type FARGATE



ecs-cli compose --project-name service-b-project --file docker-compose.yml service up --container-port 8080 --target-group-arn arn:aws:elasticloadbalancing:us-east-1:820537372947:targetgroup/service-b-tg/3a30e651d497781a
aws ecs register-task-definition --cli-input-json file://~/fargate/service-a-taskdef.json
aws ecs create-service --service-name <service_name> --task-definition arn:aws:ecs:us-east-1:820537372947:task-definition/service-a:5 --load-balancer targetGroupArn=string,loadBalancerName=string,containerName=string,containerPort=integer

