<h1>AWS IAM Credential Security and Detection</h1>

<h2>Description</h2>
In this workshop we will use an AWS environment configured with logging and analysis capabilities to detect and analyze a simulated security event based on the unauthorized use of IAM credentials. We will use AWS tools to perform various tasks that are mandatory for the security of any infrastructure using cloud technologies. I will use AWS CloudFormation to create cloud resources, Amazon Athena to review log data and other indicators of compromise, Cloudtrail to respond to activities and even use AWS CloudShell to run bash scripts and utilize AWS CLI commands to query AWS services. Linux will also play a role in this project.
<br />


<h2>Programs and Utilities Used</h2>

- <b>AWS CloudFormation</b> 
- <b>Amazon Athena and AWS CloudTrail</b>
- <b>AWS Bashshell, CLI, AWS CIRT Playbooks</b>
- <b>Amazon S3 buckets, EC2 Instances, etc</b> 

<h2>Lab Overview:</h2>

<p align="center">
  
To get this started, first we need to create a collection of AWS resources known as a 'stack' inside of AWS CloudFormation. This stack will come in the way of a template we will upload for the sake of demonstration. After uploading the stack, we need to specify the details by giving it a name (TDIR) and using today's date as the parameters. We can leave everything else default for now and submit it. After a few minutes, we get a CREATE COMPLETE in the events and that means everything went well and the environment is ready to go!  <br/>
<img src="https://i.imgur.com/Fe6ONLX.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/E37D8Nh.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/xpFdnPA.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/IZIxZU3.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Now that the environment was created, it's time to start using Amazon Athena. Amazon Athena is a serverless interactive query service that allows you to query data sets using standard SQL. Using the AWS Console Manager, we navigate to Athena and use the Query Editor to selec the appropriate workgroup (IRWorkshopAthenaWorkgroup) and the appropriate database (irworkshopgluedatabase). Running the code "SELECT * FROM "irworkshopgluedatabase"."irworkshopgluetablevpcflow" limit 10;" we receive results in a table comprised of account id, start and end time and many other factors that can help us analyze the data. The EVENTNAME field is interesting as it denotes specific actions and APIs made to the AWS Account that have been logged by CloudTrail.  <br/>
<img src="https://i.imgur.com/n9xOdct.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/wRTwHlR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Some commonly used queries in Athena are bundled with the Security Analytics Bootstrap and these can be viewed and used in the 'Saved Queries' tab of the editor. <br/>
<img src="https://i.imgur.com/DMN2o5z.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
Instead of writing our own query, let's use one of the saved queries to obtain a User Activity Summary, filtering only on high volume read-only GET/LIST/DESCRIBE calls. We need to click on the ID for the CloudTrailExampleQueries group. This should populate the Query editor with the CloudTrailExampleQueries group of queries. We are interested in lines 106 through 119 so we select them all, use ctrl +c to copy and then press the '+' in the upper right of the query editor to open a new query. In order for the query to work I need to edit lines 11 and 12 to put in my account number and the date. Running it will bring about actions taken by CloudFormation below. <br/>
<img src="https://i.imgur.com/v5ulr8K.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/tvcJ4Sc.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/0BD4R25.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Next we will simulate a scenario where exposed IAM credentials are used to access an AWS account through the use of a script invoked within AWS CloudShell. The script will perform some of the activities commonly seen by the AWS CIRT (Customer Incident Response Team) during this type of security event. From the AWS Console Manager, we can click on the appropriate icon up top to use the AWS CloudShell. AWS CLI is often used to take advantage of exposed IAM credentials. First we upload a file 'iam-credential-exposure.sh' and check to make sure it's good by using the 'ls' command. Next, in order to make it executable and run the script we input 'chmod +x iam-credential-exposure.sh' and './iam-credential-exposure.sh'. This is how we get our access id key.
 <br/>
<img src="https://i.imgur.com/Plytm2t.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/r8h4lMH.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Lastly, we will use AWS CloudTrail and Amazon Athena to investigate the access id key we obtained. CloudTrail logging was enabled earlier in the lab with CloudFormation, and allows us to view the API calls made as a result of the unauthorized use of credentials. By Using Athena to run 'SELECT * FROM "irworkshopgluedatabase"."irworkshopgluetablecloudtrail" where useridentity.accesskeyid = 'OUR NEWLY ESTABLISHED KEY' ' we can find out what time and date the exposed key was first used. By running our query again, we can use another column to locate a user associate with the key (tdir-workshop-nwolf-dev). In this instance it shows the Access Key ID was used in an attempt to retrieve the identity of the user associated with the Access Key ID itself (GetCallerIdentity), list the policies, users and roles in the account (ListPolicies, ListUsers, and ListRoles), describe all of the EC2 instances in the account (DescribeInstances), attach a user policy to a user (AttachUserPolicy), create a user (CreateUser), and create a set of access keys for a user (CreateAccessKey). The errorcode and errormessage columns indicate whether these attempted AWS API calls were successful or not. In a real investigation, looking at fields such as 'sourceip' and 'useragent' can show that applications might be used outside of your native environment.
 <br/>
<img src="https://i.imgur.com/GDvil6Q.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/bLrel4v.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://static.us-east-1.prod.workshops.aws/public/d0e1d868-7e48-4035-9801-bef71eb472cd/static/detect-02.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/nalTIUs.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/5oEwL4l.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

Thank you for looking at a brief overview of the capabilities of not only myself but the AWS platform as a whole. There are so many more tools and capabilities that can be explored and I look forward to doing more projects on them. Thank you for your time!
</p>
