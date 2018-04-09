## CloudFormation Sample to create an EC2, Security Group and Installation of Apache/PHP

### Command to create/update/delete the stack
```
$ aws cloudformation create-stack --stack-name <stackname> --template-body file://ec2.sg.php.cf.json

$ aws cloudformation update-stack --stack-name <stackname> --template-body file://ec2.sg.php.cf.json
```
Note: By default, the cfn-hup daemon runs every 15 minutes, so it may take up to 15 minutes for the application to change once the stack has been updated.

### CloudFormation Helper Scripts
Learn more about cfn-* scripts:
* https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-helper-scripts-reference.html
* https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/updating.stacks.walkthrough.html#updating.create.initial.stack

Learn more about Bootstrapping Application with CloudFormation
* https://s3.amazonaws.com/cloudformation-examples/BoostrappingApplicationsWithAWSCloudFormation.pdf

### Metadata
```json
"Metadata" : {
         "AWS::CloudFormation::Init" : {
                        "configSets" : {
                            "webapp_install" : ["install_cfn", "install_webserver"]
                        },
```
The Metadata here creates a CloudFormation Init key that will be used by the CloudFormation service. In the Init key you can define a configSet, which can be thought of as a list of functions to run. Each function will be declared below and will list actions to be performed within them. From the code above you can tell that the webapp_install configSet is made up of two additional sections to be processed: install.cfn and install_webserver.



