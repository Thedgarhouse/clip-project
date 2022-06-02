# clip-project

Devops challenge for Clip interview

The following project is meant to create and deploy using AWS CloudFormation the infrastructure and application
specified by the Clip Devops challenge.

The challenge stated the following requirements:

1. **Deploy infrastructure using IaC:** The diagram showed an EC2 instance and an Aurora MySQL instance being deployed
   each in their own subnet
2. **Create a schema in the MySQL database:** The schema showed a table named "pet" with attributes for name, owner,
   species and sex, most of them as varchar(20), with sex being the exception as char(1)
3. **Create an application:** This application would act as an API exposing an endpoint for "/api/pet/(name)" and it
   would return the data for name, owner and specie of the pet

There were also some general points to cover:

1. The app will be hosted in a public Github repository
2. A script must be provided in order to deploy the IaC
3. A script must be provided in order to deploy the application/API
4. The EC2 machine will need to be upgraded regularly

With that in mind, the following project was created.

This project contains a simple YAML file to be used with AWS CloudFormation for the deployment of the infrastructure as
well as the application detailed previously. The project also includes a parameters.json file that contains the
parameters that will be needed for deploying the CloudFormation template.

The CloudFormation template will start by deploying the two subnets, which requires a VPC to be specified. Afterwards,
it will deploy an Aurora RDS Cluster in order to be able to deploy an Aurora MySQL DB instance. The design diagram
showed a singular DB instance in a singular subnet, but in order to deploy an Aurora instance it is required to have an
Aurora Cluster, which in turn requires a DB Cluster, which requires a DB Subnet Group, which requires at least two
subnets to be specified. The subnets used for this subnet group are the previously mentioned subnets, and for this
reason the subnet in which the Aurora Instance will be deployed cannot be determined, but it is guaranteed to be in
either of those subnets.

Afterwards, an EC2 Key Pair is created to subsequently deploy an EC2 instance. This EC2 instance contains UserData to
notify AWS CloudFormation when the EC2 instance has finished running its commands. It also contains an
AWS::Cloudformation::Init declaration for installing the mysql packages, creating the sql file, and executing the sql
script. After the EC2 instance has sent a signal of completion the CloudFormation stack can continue.

Finally, the stack will create an IAM role for Lambda execution, and a Lambda function that will work as the API. The
function is written in Python, and it contains a Flask app to serve the API route. The app will connect to the Aurora
instance and retrieve the needed data.

For deploying the CloudFormation template you can run the following command:

`aws cloudformation create-stack --stack-name "Clip-test" --template-body file://.\clip_infrastructure.yaml --parameters file://.\parameters.json --capabilities CAPABILITY_IAM`

**Extra Points:**
1. This application can be exposed to the internet by either using a public endpoint on the Lambda or by creating an AWS
API Gateway and using the Lambda integration with it.
2. This application can be dynamically scaled. The Aurora cluster helps with the elasticity of the database, allowing
for the creation of multiple replicas if needed. AWS Lambda by itself also allows elasticity. The EC2 instance could be
deployed with an Auto Scaling Group (ASG) using a Launch Configuration, while defining metrics for scaling of the ASG,
such as network traffic or memory usage. The subnets could also be deployed with a bigger CIDR block if more IPs are
needed in the future.
3. We can create in our setup SQL script user credentials for a developer and then share it with them. The Aurora
instance can be accessed either through its public endpoint if available. These credentials can be configured so that
the user can only access specific tables in the database.