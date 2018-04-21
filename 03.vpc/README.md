## CloudFormation Sample to create VPC website with two subnets. 

### Command to create/update/delete the stack
```
$ aws cloudformation create-stack --stack-name myAWSInfrastructure --template-body file://01.vpc.json

$ aws cloudformation update-stack --stack-name myAWSInfrastructure --template-body file://02.elb.json --parameters ParameterKey=VPCID,ParameterValue=vpc-86b32fe1  ParameterKey=SubnetId,ParameterValue=vpc-subnet-7437412f ParameterKey=SecurityGroupId,ParameterValue=sg-fa919f83
```


### Create VPC networking
A VPC spans all the Availability Zones in the region. After creating a VPC, two subnets are created in Availability Zones. Each subnet must reside entirely within one Availability Zone and cannot span zones. To check the created subnet's Availability Zones, you can go to VPC dashboard and click "Subnets". The Availability Zone shows one column of the subnet's information

