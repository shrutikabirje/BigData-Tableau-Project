import json
import os
import boto3

def lambda_handler(event, context):

    s3_client = boto3.client('s3')
    
    cf_client = boto3.client('cloudformation')
    
    template_data = s3_client.get_object(Bucket=os.getenv('S3_BUCKET'),Key=os.getenv('S3_KEY'))
    
    template_data_body = template_data['Body'].read(template_data['ContentLength']).decode('utf-8')
    
    cloud_formation_stacks = cf_client.list_stacks()['StackSummaries']
    
    cloud_formation_stack_names = [stack['StackName'] for stack in cloud_formation_stacks]
    
    if event['stack_name'] in cloud_formation_stack_names:
    
        cf_client.delete_stack(StackName=event['stack_name'])
        
    cf_client.create_stack(StackName=event['stack_name'], TemplateBody=template_data_body, Capabilities=['CAPABILITY_IAM'], OnFailure='DO_NOTHING')
    
    return {
        'statusCode': 200,
        
        'body': json.dumps('Congrats EMR Created!!!')
        
    }
