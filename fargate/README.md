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

Choose at least 2 subnets to set as environment variables.  These will be used to populate the ecs-params.yml file.

```
export SUBNET_ID_1=<subnet_id_1>
export SUBNET_ID_2=<subnet_id_2>
```

Create a security group. Replace *<group_name>*, *<description_text>*, and *<vpc_id>* with the appropriate values. The *<vpc_id>* should match the vpc id you used earlier. 

```
export SG_ID=$(aws ec2 create-security-group --group-name <group_name> --description <description_text> --vpc-id <vpc_id> | jq -r '.GroupId')
```

Add the following inbound rules to the security group.

```
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol all --port all --source-group $SG_ID
```

Create an Application Load Balancer (ALB), listener, and target group for service B.

```
export LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer --name <load_balancer_name> --subnets $SUBNET_ID_1 $SUBNET_ID_2 --security-groups $SG_ID --scheme internet-facing --type application | jq -r '.LoadBalancers[].LoadBalancerArn')
export TARGET_GROUP_ARN=$(aws elbv2 create-target-group --name <target_group_name> --protocol HTTP --port 8080 --vpc-id <vpc_id> | jq -r '.TargetGroups[].TargetGroupArn')
aws elbv2 create-listener --load-balancer-arn $LOAD_BALANCER_ARN --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=$TARGET_GROUP_ARN
```

Get the DNS name of the load balancer. 

```
export SERVICE_B_ENDPOINT=$(aws elbv2 describe-load-balancers --load-balancer-arn $LOAD_BALANCER_ARN | jq -r '.LoadBalancers[].DNSName')
```

launch service-b
ecs-cli compose service up --deployment-max-percent 100 --deployment-min-healthy-percent 0 --load-balancer-name service-b-lb \
--target-group-arn arn:aws:elasticloadbalancing:us-east-1:820537372947:targetgroup/service-b-tg/3a30e651d497781a --launch-type FARGATE



ecs-cli compose --project-name service-b-project --file docker-compose.yml service up --container-port 8080 --target-group-arn arn:aws:elasticloadbalancing:us-east-1:820537372947:targetgroup/service-b-tg/3a30e651d497781a
aws ecs register-task-definition --cli-input-json file://~/fargate/service-a-taskdef.json
aws ecs create-service --service-name <service_name> --task-definition arn:aws:ecs:us-east-1:820537372947:task-definition/service-a:5 --load-balancer targetGroupArn=string,loadBalancerName=string,containerName=string,containerPort=integer

