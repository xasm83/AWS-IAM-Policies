AWS-IAM-Policies
================

Example of AWS IAM policy to restrict and isolate AWS users using tags and naming conventions.

This is a rough and non precise attempt to evaluate IAM. You will need to refine and test it in case you need to apply the policy to PROD!

The policy used tags to protect users from other users for tagable resources. The main problem is that AWS tags instances after creation and it does not tag all underlying resources used by the instance. The other problem is that tags are supported only for a subset of AWS resources. For example ELB, Elasticbeabstalk,S3 do not support tags for IAM. Basicly the only thing which is more or less covered by tags is EC2(and even there not all actions are covered by IAM). Here is a more precise list for services which support tags to a some degree: http://docs.aws.amazon.com/IAM/latest/UserGuide/Using_SpecificProducts.html. Also elastic ips and keypairs are not restrictable at all. Some resources are created on behalf(in Wizards) of AWS and it is not possible  to control names and so to use naming convention to control permissions(s3buckets created elasticbeanstalk - had to allow it for beanstalk completely). 

So for EC2 if a user want to protect his specific resource he should tag it with username tag and specify his username as a value.  Once the resource is tagged only users with matching username can perform sensitive operations. Which are:

        "ec2:Delete*",
        "ec2:Detach*",
        "ec2:Revoke*",
        "ec2:TerminateInstances",
        "ec2:RebootInstances",
        "ec2:StopInstances",
        "ec2:StartInstances"


By default tags do not work - that is done for convenience as I believe there is only a small subset of cases when you want to protect resources in comparison to normal routine cases.        

The other resources like S3,ELB,Elasticbeanstalk use naming convention for protection and it is turned on by default. So for S3 when you operate on any resource where you can specify name always use username_xxx pattern, where xxx is the part you can adjust. E.g users are only allowed to create username_mybucket1233 buckets. username1 will not be able to delete username_mybucket1233 bucket. S3 is restricted by region us-east-1. S3 covered by IAM very poor in general.

For ELB the convention is to  use usernamexxx where xxx ELB name when you create ELBs. ELBs are restricted by us-east-1 region.


For Elasticbeanstalk usernamexxx convention is used for all actions so users cannot manage environments and applications of other users. Region is enforced for Elasticbeanstalk operations. It is not possible to limit  instance size there. And I do not see how to use existing instances or force users to use existing instances. Users are restricted to do a sensetive actions on environments/applications named not bound to their name. All underlying EC2 resources should be tagged for protection - eg in case  user tries to terminate instance directly instead of using environment/application delete action which is protected by a naming convention.

So basically use naming convention for everything, tag all sensetive EC2 resources(security groups, instances, etc) to manage permissions. If you or aws console cant perform an operation on a resource check that you are using the right name everywhere or that all underlying resources are tagged accordingly(or remove tags). List and describe operations are enabled as AWS console uses it everywhere and generates lots of errors if it is not enabled.
