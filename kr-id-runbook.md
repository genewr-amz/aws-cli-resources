# Immersion Day RunBook

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

## Get Log Group Name
First you'll need to get the unique name for the CloudWatch log group in the target AWS account 
Run the following CLI command to get the lognoam and store it as environment variable  cw_log

```bash
export cw_log=$(aws logs describe-log-groups --log-group-name-prefix kr- --query 'logGroups[*].logGroupName' | sed 's/[]    "[]//g')   

```
enter the following command to see the log name:
```bash
echo $cw_log
```

## Begin Investigation


#### IAM Access Denied Attempts

1. Open the **iam-access-denied.json** located in the **iam-queries** folder. 

    Update the **logGroupName** and **Endtime** values, then save the file.
    To get the current endime period you can run the following command:
    
    ```bash
    date +%s
    ```
    
    Sample of the json content:
    ```json
    {
        "logGroupName": "kr-id-smoke-test-v4-CloudWatchLogGroup-QKWd8UPfvtET",
        "startTime": 1617788538, 
        "endTime": 1618243127, 
        "queryString": "filter errorCode like /Unauthorized|Denied|Forbidden/ | fields awsRegion, userIdentity.arn, eventSource, eventName, sourceIPAddress, userAgent", 
        "limit": 1000
    }
    
    ```

2. Run The following command to start the query 

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

2. Retreive the results. 

    ```bash
    aws logs get-query-results --query-id "5ad84eaf-85b0-43af-81a3-0713b105d659" #<<-- Your query ID will be diffrent
    ```
    This returns the requls in a json format which is great for using as input into another tool but difficut for humand to comprenend. 
    
    If you are doing an ad-hoc investigation us this command instead
    
    ```bash
    aws logs get-query-results --query-id "5ad84eaf-85b0-43af-81a3-0713b105d659" --output table
    ```
    
3. Look at a specific log record
    
    If you want to review specifics of a particulare eventName, copy the **@ptr** value and paste it in teh followign command 
    
    ```bash
    aws logs get-log-record --log-record-pointer "CoEBCkQKQDkwNzU2NjgwOTgyODprci1pZC1zbW9rZS10ZXN0LXY0LUNsb3VkV2F0Y2hMb2dHcm91cC1RS1dkOFVQZnZ0RVQQAxI5GhgCBd4EB18AAAAEwmNzFgAGB0YEwAAAAPIgASjY6+K0jC8w0ZHjtIwvOJ0BQMqsDUiu8AZQl7UGEEIYAQ==" --output table
    
    # or
    
    aws logs get-log-record --log-record-pointer "CoEBCkQKQDkwNzU2NjgwOTgyODprci1pZC1zbW9rZS10ZXN0LXY0LUNsb3VkV2F0Y2hMb2dHcm91cC1RS1dkOFVQZnZ0RVQQAxI5GhgCBd4EB18AAAAEwmNzFgAGB0YEwAAAAPIgASjY6+K0jC8w0ZHjtIwvOJ0BQMqsDUiu8AZQl7UGEEIYAQ==" --query 'logRecord[*].eventTime
    ```
    