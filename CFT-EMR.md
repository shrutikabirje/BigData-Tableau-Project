AWS CloudFormation provides users with a simple way to create and manage a collection of Amazon Web Services (AWS) resources by provisioning and updating them in a predictable way. AWS CloudFormation enables you to manage your complete infrastructure or AWS resources in a text file.

In simple terms, it allows you to create and model your infrastructure and applications without having to perform actions manually.

Steps for creating CFT:

1.Create a new template or use an existing CloudFormation template using the JSON or YAML format.

2.Save your code template locally or in an S3 bucket.

3.Use AWS CloudFormation to build a stack on your template.

4.AWS CloudFormation constructs and configures the stack resources that you have specified in your template.

```

{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Parameters" : {
    "InstanceType" : {
	  "Default" : "m5.xlarge",
      "Type" : "String"
    },
    "ReleaseLabel" : {
	  "Default": "emr-5.32.0",
      "Type" : "String"
    },
    "SubnetId" : {
	  "Default":"subnet-0040c866",
      "Type" : "String"
    },
    "TerminationProtected" : {

      "Type" : "String",
      "Default" : "false"
    },
    "ElasticMapReducePrincipal" : {
	  "Default" : "elasticmapreduce.amazonaws.com",
      "Type" : "String"
    },
    "Ec2Principal" : {
	  "Default" : "ec2.amazonaws.com",
      "Type" : "String"
    },
	"EMRLogDir": {
            "Description": "Log Dir for the EMR cluster",
			"Default" : "s3://my-emr-logs/elasticmapreduce/",
            "Type": "String"
        },
	"KeyName": {
            "Description": "Name of an existing EC2 KeyPair to enable SSH to the instances",
	        "Default" : "cdac_project",
            "Type": "String"
        }
  },
  "Resources": {
    "cluster": {
      "Type": "AWS::EMR::Cluster",
      "Properties": {
      "Applications":[{
                        "Name": "Hadoop"
                    },
					{
                        "Name": "Hive"
                    },
					{
                        "Name": "Spark"
                    },
					{
                        "Name": "ZooKeeper"
                    }
					],
				
        "Instances": {

          "MasterInstanceGroup": {
            "InstanceCount": 1,
            "InstanceType": {"Ref" : "InstanceType"},
            "Market": "ON_DEMAND",
            "Name": "Master"
          },
            
         "CoreInstanceGroup": {
            "InstanceCount": 1,
            "InstanceType": {"Ref" : "InstanceType"},
            "Market": "ON_DEMAND",
            "Name": "Core"
          },
          "TerminationProtected" : {"Ref" : "TerminationProtected"},
          "Ec2SubnetId" : {"Ref" : "SubnetId"},
		  "Ec2KeyName": {"Ref": "KeyName"}
        },
		"LogUri": {
                    "Ref": "EMRLogDir"
                },
        "Name": "USAccidents",
        "JobFlowRole" : {"Ref": "emrEc2InstanceProfile"},
        "ServiceRole" : {"Ref": "emrRole"},
        "ReleaseLabel" : {"Ref" : "ReleaseLabel"},

        "VisibleToAllUsers" : true,
        "Tags": [
          {
            "Key": "key1",
            "Value": "value1"
          }

        ]
      }
    },
    "emrRole": {
      "Type": "AWS::IAM::Role",
      "Properties": {
        "AssumeRolePolicyDocument": {
          "Version": "2008-10-17",
          "Statement": [
            {
              "Sid": "",
              "Effect": "Allow",
              "Principal": {
                "Service": {"Ref" : "ElasticMapReducePrincipal"}
              },
              "Action": "sts:AssumeRole"
            }
          ]
        },
        "Path": "/",
        "ManagedPolicyArns": ["arn:aws:iam::aws:policy/service-role/AmazonElasticMapReduceRole"]
      }
    },
    "emrEc2Role": {
      "Type": "AWS::IAM::Role",

      "Properties": {
        "AssumeRolePolicyDocument": {
          "Version": "2008-10-17",
          "Statement": [
            {
              "Sid": "",

              "Effect": "Allow",
              "Principal": {
                "Service": {"Ref" : "Ec2Principal"}
              },
              "Action": "sts:AssumeRole"
            }
          ]
        },
        "Path": "/",
        "ManagedPolicyArns": ["arn:aws:iam::aws:policy/service-role/AmazonElasticMapReduceforEC2Role"]
      }
    },
    "emrEc2InstanceProfile": {
      "Type": "AWS::IAM::InstanceProfile",
      "Properties": {
        "Path": "/",
        "Roles": [ {
          "Ref": "emrEc2Role"
        } ]
      }
    },

     "SparkStep": {
            "Properties": {
                "ActionOnFailure": "CONTINUE",
                "HadoopJarStep": {
                    "Args": [
                        "spark-submit",
                        "--deploy-mode",
                        "cluster",
                        "--conf",
                        "spark.sql.catalogImplementation=hive",
                        "s3://us-accident/project_script/us_accidents.py"
                    ],
                    "Jar": "command-runner.jar"  
                },
                "JobFlowId": {
                    "Ref": "cluster"
                },
                "Name": "SparkStep"
            },
            "Type": "AWS::EMR::Step"
        }
	 
  }
}

```
