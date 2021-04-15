# Workshop environment setup

This is the first section of the AWS KMS Workshop where we will get things ready to start with the workshop.
Please ensure you have read and understood the [prerequisites for the workshop](https://github.com/charliejllewellyn/aws-kms-workshop#pre---requisites). 

Especially, though AWS KMS prior knowledge is not really needed, Workshop is more meaningful if you take a look at this introduction to AWS KMS:

* [What is Key Management Service?](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
* [Getting Started with AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/getting-started.html)

---

When you are ready, please follow the following steps to create all the artifacts we will be using during the workshop:


1. Login into your AWS account and navigate to the region you want work in. 



2. Download the Workshop's **CloudFormation template on the (Save Link As) [following link](https://raw.githubusercontent.com/charliejllewellyn/aws-kms-workshop/master/res/cf-workshoptemplate.txt)**. This template will create a Role named "**KMSWorkshop-InstanceInitRole**" and an Amazon S3 bucket named "**kmsworkshop-accountid**", where accountid is the identifier of your account.
 

   Go to the AWS Console, navigate to "**CloudFormation**" Service and select "**Create Stack**" as you can see in figure below:
   
   
   
![alt text](/res/S0F1.png)
   
   
3. Then, in the "**Specify Template**" area, select "**Upload Template**" and browse for the template we downloaded just        before. Click "**Next**" and give the stack a name, like "KMSWorkshop-Stack". Hit "**Next**". Leave the default values that appear in this new page and hit "**Next**" again. In this new page, make sure you click the checkbox "**The following  
   resource(s) require capabilities: [AWS::IAM::Role]**" at the botton, and click "**Create Stack**". 
   
   The stack is now being created. If you got lost in the process, please look into the [CloudFormation Stack Creation documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)
   

4. Once the CloudFormation Stack is Ready, head over to the EC2 console and connect the instance **KMS-Workshop** using Sessions Manager. If you need help, [check the options here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/session-manager.html).


If you can connect to your instance then **You should now be ready to start with the workshop**, let's [Go to first section of workshop](https://github.com/charliejllewellyn/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md)


