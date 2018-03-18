aws iam get-role --role-name xray-role-for-fargate --query 'Role.Arn'
aws iam get-role --role-name ecsTaskExecutionRole --query 'Role.Arn'
aws ec2 describe-subnets --query 'Subnets[?VpcId==`vpc-7bc1da1d`].SubnetId'
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
