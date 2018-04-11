## CloudFormation Sample to create WordPress website with Load Balancer. This template installs HA, scalable WordPress deployment using a multi-AZ AWS RDS DB.

### Command to create/update/delete the stack
```
$ aws cloudformation create-stack --stack-name <stackname> --template-body file://ec2.wordpress.json --parameters ParameterKey=KeyName,ParameterValue=gtixcaas 

$ aws cloudformation update-stack --stack-name <stackname> --template-body file://ec2.wordpress.json --parameters ParameterKey=KeyName,ParameterValue=gtixcaas 
```
Note: By default, the cfn-hup daemon runs every 15 minutes, so it may take up to 15 minutes for the application to change once the stack has been updated.


### cfn-signal & WaitCondition
```
	"UserData" : { "Fn::Base64" : { "Fn::Join" : ["", [
		"#!/bin/bash -xe\n",
		"yum update aws-cfn-bootstrap\n",
		"/opt/aws/bin/cfn-init ",
		" --stack ", { "Ref" : "AWS::StackName" },
		" --resource LaunchConfig ",
		" --configsets wordpress_install ",
		" --region ", { "Ref" : "AWS::Region" }, "\n",
		"/opt/aws/bin/cfn-signal -e $? '", { "Ref" : "WebServerWaitHandle" }, "'\n"
	]]}}
```
```
	"WebServerWaitHandle" : {
      "Type" : "AWS::CloudFormation::WaitConditionHandle"
    },
    "WebServerWaitCondition" : {
      "Type" : "AWS::CloudFormation::WaitCondition",
      "DependsOn" : "WebServerGroup",
      "Properties" : {
        "Handle" : {"Ref" : "WebServerWaitHandle"},
        "Timeout" : "1200"
      }
    },
```

To wait for the application to be ready, we can create a WaitCondition and a WaitConditionHandle. Then use the cfn-signal helper script to signal that the application is installed. The cfn-signal script is a wrapper that makes it easy to signal a wait object from your bootstrap script. 
We can think of a wait condition as a timed semaphore. When the WaitCondition resource is created, a timer is started. The WaitCondition becomes CREATE_COMPLETE after it receives a signal. If no signal is received before the timeout, the WaitCondition enters the CREATE_FAILED state and the stack creation is rolled back.
