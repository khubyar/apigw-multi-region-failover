# Amazon API Gateway Multi-Region Public REST API Failover: Service 2


Service2 consists of a Regional rest API with a single root path calling a Lambda function.

This stack deploys service2 on a primary and secondary Region. It will also setup Route 53 public failover records and Route 53 ARC routing controls for you.

## Deployment Instructions

1. Change directory to:
    ```
    cd apigw-multi-Region-failover/service2
    ```
1. From the command line, use AWS SAM to deploy the AWS resources for the stack as specified in the template.yml file on the primary Region:
    ```
    sam deploy --guided --config-env primary
    ```
1. During the prompts:
    * **Stack Name:** Enter a stack name.
    * **AWS Region:** Enter the desired primary AWS Region. This stack has been tested with both us-east-1 and us-east-2.
    * **PublicHostedZoneId:** You must have a public hosted zone in Route 53 with your domain name (i.e. example.com). Enter the Hosted Zone Id for this hosted zone.
    * **DomainName:** Enter your custom domain name (i.e. service2.example.com).
    * **CertificateArn** You must have an ACM certificate that covers your custom domain namespace (i.e. *.example.com) on the Region your are deploying this stack. Enter the ARN for this certificate here. **Make sure you are getting the certificate ARN for the right Region**.
    * **Route53ArcClusterArn:** Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the Route 53 Cluster ARN created during that deployment.
    * **Service2ControlPlaneArn**: Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the  Route 53 ARC control pane ARN for service 2.
    * **Stage:** Enter the name of the stage within your API Gateway that you would like to map to your custom domain name.
    * **FailoverType:** Accept the defaults and use **PRIMARY** here.
    * Allow SAM CLI to create IAM roles with the required permissions.
    * Allow SAM CLI to create the Service2LambdaRegionalApi Lambda function.
    * **SAM configuration environment** Accept the **primary** default value.

    Once you have run `sam deploy --guided --config-env primary` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy --config-env primary` in future to use these defaults.

1. From the command line, use AWS SAM to deploy the AWS resources for the stack as specified in the template.yml file on the primary Region:
    ```
    sam deploy --guided --config-env secondary
    ```
1. During the prompts:
    * **Stack Name:** Enter a stack name.
    * **AWS Region:** Enter the desired secondary AWS Region. This stack has been tested with both us-east-1 and us-east-2. **Make sure to use a different Region from the primary one**.
    * **PublicHostedZoneId:** You must have a public hosted zone in Route 53 with your domain name (i.e. example.com). Enter the Hosted Zone Id for this hosted zone.
    * **DomainName:** Enter your custom domain name (i.e. service2.example.com).
    * **CertificateArn** You must have an ACM certificate that covers your custom domain namespace (i.e. *.example.com) on the Region your are deploying this stack. Enter the ARN for this certificate here. **Make sure you are getting the certificate ARN for the right Region**.
    * **Route53ArcClusterArn:** Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the Route 53 Cluster ARN created during that deployment.
    * **Service2ControlPlaneArn**: Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the  Route 53 ARC control pane ARN for service 2.
    * **Stage:** Enter the name of the stage within your API Gateway that you would like to map to your custom domain name.
    * **FailoverType:** Accept the defauls and use **SECONDARY** here.
    * Allow SAM CLI to create IAM roles with the required permissions.
    * Allow SAM CLI to create the Service2LambdaRegionalApi Lambda function.
    * **SAM configuration environment** Accept the **primary** default value.

    Once you have run `sam deploy --guided --config-env secondary` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy --config-env secondary` in future to use these defaults.
    
1. Note the outputs from the SAM deployment process. These contain details which are used for testing.

## How it works

This stack will deploy an Amazon API Gateway REST Regional API with a Lambda integration. The AWS Lambda function is written in Python3.9. The function returns a small message with the service name and the Region it is deployed in. The inline code of the Lambda function is written in the template itself.

## Testing

Once the stack is deployed, get the API endpoint from the EndpointUrl output parameter.
Paste the URL in a browser, or in Postman, or using the curl command.
Eg: 
```bash
curl https://aabbccddee.execute-api.us-east-1.amazonaws.com/prod
```

You should see a response similar to:
```json
{"service": "service2", "region": "your-selected-region"}
```


Now test that one of your Regional services is accessible via your custom domain.
You can get that URL from the **CustomDomainNameEndpoint** output parameter.
Eg: 
```bash
curl https://service2.example.com
```

You should see a response similar to:
```json
{"service": "service2", "region": "your-primary-region"}
```

## Cleanup
 
1. Delete the stack on the primary Region.
    ```bash
    sam delete --config-env primary
    ```
1. Delete the stack on the secondary Region.
    ```bash
    sam delete --config-env secondary
    ```
----
Copyright 2023 Amazon.com, Inc. or its affiliates. All Rights Reserved.

SPDX-License-Identifier: MIT-0
