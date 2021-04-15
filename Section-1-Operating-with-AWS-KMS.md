# Operating with AWS KMS and CMKs

In this first section we are going to learn the core operations of AWS KMS, that would allow us to go deeper into the service and its best practices. The section has four main areas:
 * [Creating Customer Master Keys (CMK)](https://github.com/charliejllewellyn/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md#creating-customer-master-keys-cmk)
 * [Rotating AWS KMS CMKs](https://github.com/charliejllewellyn/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md#rotating-AWS-KMS-CMKs)
 * [Deleting AWS KMS CMKs](https://github.com/charliejllewellyn/aws-kms-workshop/blob/master/Section-1-Operating-with-AWS-KMS.md#deleting-AWS-KMS-CMKs)
----


## Creating Customer Master Keys (CMK)

CMKs are the primary resources in AWS KMS. You can use a CMK to encrypt and decrypt up to 4 kilobytes (4096 bytes) of data. However, most commonly, you will use CMKs to generate, encrypt, and decrypt the [data keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) that you use outside of AWS KMS to encrypt your data.

There are different types of CMKs in KMS, [see documentation here](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys). 
In this section we will create a CMK with key material coming from AWS KMS, and later we will generate a CMK with your own key material. It is important to remember that CMKs never leave AWS KMS unencrypted.

### Step 1 - create CMKs

Time to get hands-on with KMS. **Connect to the instance created via terminal** to start working with the CMKs.

In order to create our first CMK, we will use the [AWS CLI](https://aws.amazon.com/cli/) in the instance. We will use it, instead of the AWS console, beacuse it will provide you with deeper insights and understanding of the process. Once you understand it, creating CMKs from the console will be a breeze.

To use the AWS CLI you might need to configure your region in AWS CLI first. You can do so typing the command "**aws configure**" in the instance once you are connected via terminal. 
Leave all fields blank except for the default region. Choose the code of the region you are working in ([region codes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions)) and set it as default region in AWS CLI. 

See an example below setting up Ireland as the default region in AWS CLI:

```
[ec2-user@ip-10-0-X-X ~]$ sudo -i
[ec2-user@ip-10-0-X-X ~]$ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: eu-west-2
Default output format [None]: 
[ec2-user@ip-10-0-X-X ~]$ 
```

More information can be found [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration).

Once AWS CLI is ready, try typing the following command to create the key:

```
$ aws kms create-key
```


The response from above command should be an error message like the one below. 

```
An error occurred (AccessDeniedException) when calling the CreateKey operation: User: arn:aws:sts::account-id:assumed-role/KMSWorkshop-InstanceInitRole/instanceid is not authorized to perform: kms:CreateKey on resource:
```

This is because the initial role we have assigned to the instance does not include the capability to create keys. We need to add a policy to the role in order to enable us to perform certain actions with AWS KMS during the workshop. 

Following **least privilege best practices**, we will be attaching policies with permissions as needed for the different AWS KMS operations we are working with. In this way it is easy track and detach the policies from the role, once they are not needed, and keep the least privilege principles.



### Step 2 - Create Policy and Attach it to the role

We can add the needed permissions via the CLI or the AWS console. We will use the AWS console for this operation.

Logging into the AWS console, [click here](https://console.aws.amazon.com/?nc2=h_m_mc), and navigate to the IAM service. Then click on "**Roles**", left area of the screen.
![alt text](/res/S1F1%20IAM.png)
<**Figure-1**>


Search for the role that has been set up and attached to the instance by the CloudFormation template, its name is **KMSWorkshop-InstanceInitRole**. 

![alt text](/res/S1F2%20KMSinitRole.png)

<**Figure-2**>


Click on the role and then on "**Attach Policies**" button, we are going to provide permissions so the instance can create Keys. A new screen where you can now search for Policies will appear.
![alt text](/res/S1F3%20AttachPolicy.png)

<**Figure-3**>


Now, search "**AWSKeyManagement**", and select the policy "**AWSKeyManagementSystemPowerUser**".  That is the policy we are going to use for the instance role. **Please note**, the assigment of KMS Power User permissions is **just** for the initial walk-through in KMS, a typical user might not need the whole set of permissions. Later in the workshop we will work on how to implement more fine grained "Least Privilege" access, according to best practices,  in order to assign appropriate permissions to users and roles into KMS operations.
As this point we can review the policy, just by clicking on the small black arrow close to the policy name to expand it.

![Figure-4](/res/S1F4%20KMSPowerUserPolicy.png)

<**Figure-4**>



 See the operations the policy allowing as a Poweruser into AWS KMS.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:CreateAlias",
                "kms:CreateKey",
                "kms:DeleteAlias",
                "kms:Describe*",
                "kms:GenerateRandom",
                "kms:Get*",
                "kms:List*",
                "kms:TagResource",
                "kms:UntagResource",
                "iam:ListGroups",
                "iam:ListRoles",
                "iam:ListUsers"
            ],
            "Resource": "*"
        }
    ]
}
```


Select the policy and click the "Attach policy" button at botton right of the page. The role attached to the instance is modified with his new policy and we should be able to create a Customer Master Key (CMK) now from the instance. Let's try it.



### Step 3 - Create the key  - again - and set an alias

Run the aws kms create-key command again and this time you will get the result from the creation of the key as a JSON block with the metadata of the key. See example below:

```
$ aws kms create-key

{
    "KeyMetadata": {
        "Origin": "AWS_KMS", 
        "KeyId": "your-key-id", 
        "Description": "", 
        "KeyManager": "CUSTOMER", 
        "Enabled": true, 
        "KeyUsage": "ENCRYPT_DECRYPT", 
        "KeyState": "Enabled", 
        "CreationDate": 1538497627.686, 
        "Arn": "arn:aws:kms:eu-west-1::key/", 
        "AWSAccountId": "your-account-id"
    }
}

```
There are important fields in the metadata response. First, The **KeyId** is very relevant as it is the unique identifier of the CMK within AWS KMS. It is in the form of five blocks of digits. Take good note of the KeyId as you will be using it during the workshop extensively. Also, along the Workshop, I will reference its value as "**your-key-id**". 
Please change it for your corresponding KeyId value when applicable.

The ARN of the key and its status ("Enabled", "Disabled") are highly relevant too. Other type of information is displayed. For example: the Account Id where the key belongs to. Another relevant piece of information is the the target key usage, this is: encrypt and decrypt in this case. 

If you go back to the AWS console and navigate to the KMS service. The key you have just created is already listed there. However, as we used the create-key command without parameters, it does not contain any alias to display and looks like its alias is empty. See image below the region selection and the blank alias name.

![Figure-15](/res/S1F15.png)
<**Figure-15**>


Key aliases are very useful. They are easier to remenber when operating with keys. Most importantly, when rotation keys, as we will see later in this section, we will not have to update our code to update the new KeyIDs or ARN references. By using alias in our code to call the CMKs by them, and updating the alias CMKs to point to the newly generated key, the amount of change in our code gets minimized.

Let's create an alias, "**FirstCMK**",  with the command aws kms create-alias. 
Remember to replace 'your-key-id' with the value obtained from previous command (aws kms create-key).


```
$ aws kms create-alias --alias-name alias/FirstCMK --target-key-id 'your-key-id'
```

If you look now in the console, the CMK you just created displays now the right alias. 

![Figure-5](/res/S1F5%20Alias.png)
<**Figure-5**>


When you create the CMK from the console, just by clicking the button "create key" there are other parameters you need to set like tags, key administrators and usage permissions. This steps will basically create a Key Policy and attach it to the key together with the tags you have set. 
For the workshop, we will see how creating CMKs, policies and tags can be done from the CLI to have greater insights on their scope and implications. 


----

## Rotating AWS KMS CMKs

Key rotation is a very important in key management and a security best practice.
In AWS KMS there are different ways to rotate keys according to the way they were created.

### Step 1 - CMKs generated with AWS key material 

For CMKs created with AWS key material, you can opt-in to automatically rotate the key every year
AWS KMS generates new cryptographic material for the CMK every year. In this case, AWS KMS also saves the CMK's older cryptographic material, so it can be used to decrypt data that it encrypted.

Automatic key rotation preserves the properties of the CMK: key ID, key ARN, region, policies, and permissions, do not change when the key is rotated, so you don´t need to manually update the alias of the CMK to point to a newly generated CMK.

Let's opt-in to automatically rotate the CMK key we created before with AWS key material, remenber its alias was "**FirstCMK**", the KeyID was "**your-key-id**". 

```
$ aws kms enable-key-rotation --key-id your-key-id

```


If the command executed successfully, you have enabled the automatic rotation of the CMK, that will happen in 365 days since the command executed, this is: 1 year from now.

### Deleting AWS KMS CMKs

Deleting customer master keys is a very sensitive operation.  You should delete a CMK only when you are sure that you don't need to use it anymore. The implications of key deletion are explained in the following [section](https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html) of the AWS KMS documentation, please read carefully.

Providing the right permissions for key deletion are an important part of best practices working with AWS KMS, as we will see in next section.

If you are not sure that you need to delete the key, you might want to disable the key only. Execute the following command to change the state of our first key "**FirstCMK**" to disabled. You will have to replace "**your-key-id**" with your corresponding KeyId or ARN. (**NOTE:**) Key Aliases are not supported for this operation. 

```
$ aws kms disable-key --key-id your-key-id
```

Let's re-enable it to keep using it. In order to do so, execute the following command (again, replace "**your-key-id**" with your corresponding KeyID or ARN) :
```
$ aws kms enable-key --key-id your-key-id
```

**The commands below are for information purposes, don´t execute it as part of the workshop**. 

For the deletion operation, AWS KMS enforces a waiting period. To delete a CMK in AWS KMS you have to schedule a key deletion.
You can set the waiting period from a minimum of 7 days up to a maximum of 30 days. The default waiting period is 30 days. Let's schedule key deletion in seven days, use the following command. Please, replace "**your-key-id**" with the  corresponding KeyID or ARN for the first CMK you created with the first AWS KMS command in this workshop, the one is not currently being point at by the alias.
```
$ aws kms schedule-key-deletion --key-id your-key-id --pending-window-in-days 7
{
    "KeyId": "arn:aws:kms:your-region:your-account-id:key/your-key-id", 
    "DeletionDate": 1544659200.0
}

```

Working with CMKs that have been generated with your own key material is a bit different because you can schedule a key deletion but you can also delete key material on demand. Therefore, for deletion of key material, you can schedule a date and wait for the key material to expire or you delete it manually.

If you want to delete it **immediately**, you could issue a command like the one below to delete the key material you have imported, rendering the key unusable. You should replace "your-key-id" with your corresponding KeyID or ARN.

If for any reason you delete the key we generated with our own key material "**ImportedCMK**", later you would have to import again your key material into the CMK and into the same alias to get it back to an usable state.

## Just for information
```
$ aws kms delete-imported-key-material --key-id  your-key-id.   
```

Congratulations, you have now completed this section of the workshop. You can now go to the second section of the workshop: [Encryption with AWS KMS](https://github.com/charliejllewellyn/aws-kms-workshop/blob/master/Section-2-Encryption-with-AWS-KMS.md)


