## CloudFormation Sample to create WordPress website with Load Balancer. This template installs HA, scalable WordPress deployment using a multi-AZ AWS RDS DB.

### Command to create/update/delete the stack
```
$ aws acm list-certificates

$ aws acm request-certificate --domain-name <domain name> --domain-validation-options DomainName=<domain name>,ValidationDomain=<domain name>

$ aws cloudformation create-stack --stack-name <stackname> --template-body file://ec2.wordpress.ssl.json --parameters file://param.ssl.json 

$ aws cloudformation create-change-set --stack-name <stackname> --change-set-name <changesetname> --template-body file://ec2.wordpress.ssl.json --parameters file://param.ssl.json 

$ aws cloudformation list-change-sets --stack-name <stackname> 

$ aws cloudformation describe-change-set --stack-name <stackname> --change-set-name <changesetname> | jq '.Changes[]'

$ aws cloudformation execute-change-set --stack-name <stackname> --change-set-name <changesetname>

$ aws cloudformation update-stack --stack-name <stackname> --template-body file://ec2.wordpress.ssl.json --parameters file://param.ssl.json  
```
Note: 
1. Once #aws acm request-certificate# is issued, an approval email will be send to the domain admin, which you can find out from whois service
2. By default, the cfn-hup daemon runs every 15 minutes, so it may take up to 15 minutes for the application to change once the stack has been updated.
3. The advantage to use change set is that it gives you an oppertunity to do dry run and provides a clear audit trail

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
