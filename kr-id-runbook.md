# Immersion Day RunBook

## Situation
Today, several news outlets reported information about project Unicorn. The reported details about this project were classified and not authorized for public release. 

Leadership believes the exposed information is the the result of data breach. You have been tasked with leading an investigation to determine if and how the Unicorn project details were compromised.

Based on the information that was exposed, the breach most likely occurred within the last 24 hours, as the particular reported project details did not exist until then. 

## Mission
Your mission is to assess the AWS environment and logs for any activity that would indicate the environment has been compromised

## Execution
Commanders Intent: 
 - Identify all compromised AWS resources
 - Contain any unsecured ingress or egress points
 - Determine most likely cause of data breach

## Rules of Engagement
1. Only resources that include a tag named "flag" are in scope.
2. Once you believe you have identified the compromised resources, take action to contain the incident using any non-destructive method of your choosing.


## Check CLI
### Check Version 
```bash
aws --version
```

Version should be 2.1+ If not then upgrade:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
#Check CLI version again to confirm upgrade was successful
aws --version 
```

## Find Your Log Group Name
Amazon CloudWatch Logs can be used to monitor, store, and access your log files from many different sources including AWS CloudTrail. The following CLI command lists all your log groups so you can find yours, then enter the name you used for your log group in the next step.

```bash
export cw_log=$(aws logs describe-log-groups --log-group-name-prefix mod- --query 'logGroups[*].logGroupName' | sed 's/[]    "[]//g')   

```

enter the following command to see the log name:
```bash
echo $cw_log
```

## Begin Investigation
Navigate to the **aws-cli-resources/iam-queries** folder

In this folder you will find several CLI skeletons to assist in your investigation. 

Below is an example of how to modify and run the skeletons: 

Each Skeleton will require you includde the correct **logGroupName** and **Endtime** values, to run sucessfully.
To get the current endime period you can run the following command:
    
    ```bash
    date +%s
    ```
    
    Sample of the json content:
    ```json
    {
        "logGroupName": "mod-c551173c709a4722-CloudWatchLogGroup-3vb7wHEhC7s5",
        "startTime": 1617788538, 
        "endTime": 1618243127, 
        "queryString": "filter errorCode like /Unauthorized|Denied|Forbidden/ | fields awsRegion, userIdentity.arn, eventSource, eventName, sourceIPAddress, userAgent", 
        "limit": 1000
    }
    
    ```

Run The following command to start the query from CLI Skeleton 

    ```bash
    
    aws logs start-query --cli-input-json "file://iam-queries/iam-access-denied.json"
    ```
    
    The command will return a queryId, Save this id you will need it to retreive teh results
    
    Query ID Example:
    ```json
    {
        "queryId": "5ad84eaf-85b0-43af-81a3-0713b105d659"
    }
    ```

**Retreive The Results** 

    ```bash
    aws logs get-query-results --query-id "5ad84eaf-85b0-43af-81a3-0713b105d659" #<<-- Your query ID will be diffrent
    ```
This returns the requls in a json format which is great for using as input into another tool but difficut for humand to comprenend. 
    
   If you are doing an ad-hoc investigation us this command instead
    
    ```bash
    aws logs get-query-results --query-id "5ad84eaf-85b0-43af-81a3-0713b105d659" --output table
    ```
    
**Look at a Specific Log Record**
    
If you want to review specifics of a particulare eventName, copy the **@ptr** value and paste it in teh followign command 
    
    ```bash
    aws logs get-log-record --log-record-pointer "CoEBCkQKQDkwNzU2NjgwOTgyODprci1pZC1zbW9rZS10ZXN0LXY0LUNsb3VkV2F0Y2hMb2dHcm91cC1RS1dkOFVQZnZ0RVQQAxI5GhgCBd4EB18AAAAEwmNzFgAGB0YEwAAAAPIgASjY6+K0jC8w0ZHjtIwvOJ0BQMqsDUiu8AZQl7UGEEIYAQ==" --output table
    
    # or
    
    aws logs get-log-record --log-record-pointer "CoEBCkQKQDkwNzU2NjgwOTgyODprci1pZC1zbW9rZS10ZXN0LXY0LUNsb3VkV2F0Y2hMb2dHcm91cC1RS1dkOFVQZnZ0RVQQAxI5GhgCBd4EB18AAAAEwmNzFgAGB0YEwAAAAPIgASjY6+K0jC8w0ZHjtIwvOJ0BQMqsDUiu8AZQl7UGEEIYAQ==" --query 'logRecord[*].eventTime
    ```

**Search Events by AccessKey Id**

If you want to retreive events by AccessKeyId
    ```bash
    aws logs get-query-results --query-id "082d54b5-4a2c-4bb9-ac2e-cf1c77814878" --query 'results[*][?userIdentity.accessKeyId=='AKIA3PSXH4ENSZD64E55']' --output table
    
    ```
