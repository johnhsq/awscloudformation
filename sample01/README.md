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

### cfn-init
```
"/opt/aws/bin/cfn-init -v ",
	"         --stack ", { "Ref" : "AWS::StackName" },
	"         --resource WebServer ",
	"         --configsets webapp_install ",
	"         --region ", { "Ref" : "AWS::Region" }, "\n",
```
The *cfn-init* script uses the resource metadata that is defined in the *AWS::CloudFormation::Init* metadata key. The cfn-init helper script interprets the metadata that contains the sources, packages, files, and services. You run the script on the EC2 instance when it is launched. The script is installed by default on Amazon Linux and Windows AMIs.

### cfn-signal
```
	"/opt/aws/bin/cfn-signal -e $? ",
	"         --stack ", { "Ref" : "AWS::StackName" },
	"         --resource WebServer ",
	"         --region ", { "Ref" : "AWS::Region" }, "\n"
```
you might need to wait for an application to be deployed and running on one or more EC2 instances before the stack creation process can continue or be completed. To provide synchronization between the helper scripts on the instance and other resources in the template, AWS CloudFormation supports a WaitCondition resource. You can think of a wait condition as a timed semaphore. When the WaitCondition resource is created, a timer is started. The WaitCondition becomes CREATE_COMPLETE after it receives a signal. If no signal is received before the timeout, the WaitCondition enters the CREATE_FAILED state and the stack creation is rolled back.

### cfn-hup
```
	"services" : {
		"sysvinit" : {
		"cfn-hup" : { "enabled" : "true", "ensureRunning" : "true",
				"files" : ["/etc/cfn/cfn-hup.conf", "/etc/cfn/hooks.d/cfn-auto-reloader.conf"] }
		}
	}
```
AWS CloudFormation provides the cfn-hup helper to reconfigure, restart, or update an application on an instance as part of the stack update process. The cfn-hup helper is a daemon that takes the actions that are specified in the resource metadata after it detects changes in resource metadata. You can use the daemon to make configuration updates on your running Amazon EC2 instances through UpdateStack.
The cfn-hup helper must be configured to inspect the correct stack. This configuration is stored in the cfn-hup configuration file cfn-hup.conf.
By default, the cfn-hup daemon runs every 15 minutes, so it may take up to 15 minutes for the application to change once the stack has been updated.

### cfn-auto-reloader.conf
Dynamic Software Updates Using cfn-hup and cfn-init
By combining cfn-hup hooks with the cfn-init script, you can automatically install new versions of software when you change the metadata by updating the stack template.
```
	"/etc/cfn/hooks.d/cfn-auto-reloader.conf": {
		"content": { "Fn::Join": [ "", [
		"[cfn-auto-reloader-hook]\n",
		"triggers=post.update\n",
		"path=Resources.WebServer.Metadata.AWS::CloudFormation::Init\n",
		"action=/opt/aws/bin/cfn-init -v ",
		"         --stack ", { "Ref" : "AWS::StackName" },
		"         --resource WebServer ",
		"         --configsets webapp_install",
		"         --region ", { "Ref" : "AWS::Region" }, "\n"
		]]},          
		"mode"  : "000400",
		"owner" : "root",
		"group" : "root"
	}
```
In this file, we define a cfn-hup hook that looks for changes to the metadata that is defined in the WebServer resource in the stack and calls cfn-init if there is a change. If the metadata changes, cfn-init looks at all the versions of the packages that are defined for the WebServer resource and, if there has been a change, installs the version from the new template.

