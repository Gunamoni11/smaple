pipeline{
  agent any
  parameters{
    string (name: 'keyname', defaultValue: 'terra', description: ' ')
    string (name: 'SecurityGroup', defaultValue: 'sg-0244e3f85210cc582', description: ' ')
    string (name: 'Subnet', defaultValue: 'subnet-04bb8948', description: ' ')
            }
  stages{
       stage('hosting application'){
        steps{
          sh "ls"
          sh "aws rds create-db-instance --db-instance-identifier test-mysql-instance1 --db-name cpms --db-instance-class db.t2.micro --vpc-security-group-ids "+SecurityGroup+" --engine mysql --engine-version 5.7 --db-parameter-group-name default.mysql5.7 --publicly-accessible  --master-username admin --master-user-password Nareshkumar --allocated-storage 10 --region us-east-2"
          sleep(450)
          script{
              def cmd = "aws rds describe-db-instances --db-instance-identifier test-mysql-instance1 --region us-east-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem = readJSON text: output
              println(jsonitem)
           }
           sh "sudo sed -i.bak 's/endpoint/${jsonitem['DBInstances'][0]['Endpoint']['Address']}/g' userdata.txt"
          script{
              def cmd = "aws elbv2 create-load-balancer --name my-load-balancer --subnets "+Subnet+" subnet-15e22868 --security-groups "+SecurityGroup+" --region us-east-2 "
              def output = sh(script: cmd,returnStdout: true)
              jsonitem1 = readJSON text: output
              println(jsonitem1)
              sleep(100)
            }
          script{
              def cmd = "aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 --target-type instance --vpc-id vpc-6014600b --region us-east-2"
              def output = sh(script: cmd,returnStdout: true)
              jsonitem2 = readJSON text: output
              println(jsonitem2)
              sleep(180)
               }
           sh "aws elbv2 create-listener --load-balancer-arn ${jsonitem1['LoadBalancers'][0]['LoadBalancerArn']} --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --region us-east-2"
           sh "aws autoscaling create-launch-configuration --launch-configuration-name my-lc3-cli --image-id ami-0fb653ca2d3203ac1 --instance-type t2.micro --security-groups " + SecurityGroup + " --key-name " + keyname + " --iam-instance-profile demo-Role --user-data file://userdata.txt --region us-east-2"
           sh "aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-asg3-cli --launch-configuration-name my-lc3-cli --max-size 2 --min-size 1 --desired-capacity 1 --target-group-arns ${jsonitem2['TargetGroups'][0]['TargetGroupArn']} --availability-zones us-east-2c --region us-east-2"
        }
       }
  }
}
