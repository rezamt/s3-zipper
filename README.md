# S3 Zipper 
The following diagram shows how the overall systems work:

![](https://i.imgur.com/AHHGWbP.png)

Things to be consider:

1- Creating S3 Gateway endpoint in your VPC and allow Lambda to access to the S3 through private link

2- Lambda Configuration  
The `template.yaml` needs to be modified for the following parameters:
| Paramter Name | Default Value | Description |
| -------- | -------- | -------- |
| Runtime     | nodejs14.x     | Lambda Runtime environment     |
| MemorySize     | 512     | Lambda Memory Size     |
| Timeout     | 250     | Lambda Execution Timeout     |
| Policies     | AWSLambdaBasicExecutionRole     | Lambda Role(s)  |



```yaml
  zipperLambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/app.handler
      Runtime: nodejs14.x
      Architectures:
        - x86_64
      MemorySize: 512 # Megabyte
      Timeout: 250    # Seconds
      Description: A Lambda function that takes a list of input files from S3 bucket
      Policies:
        - AWSLambdaBasicExecutionRole  # Give Lambda execution Permission to the zipperLambda
        - AmazonS3FullAccess           # Giving Lambda Permission to S3 Bucket

```

## Deployment
To deploy this code, please make sure 
- You have configured AWS profile (key & secret) and you have access to AWS
- the AWS SAM CLI is installed and configured in your environment

```bash
set STACK_NAME=zipper
sam sync --stack-name $STACK_NAME --watch
```

## NodeJS
This project has dependencies to the following Node.js libraries:
- archiver
- stream
- request

## Testing Lambda
Once your lambda deployed you can test it using the following event sample:

```
# Sample Event for testing
{
    "bucket": "your-bucket",
    "destination_key": "zips/test.zip",
    "files": [
        {
            "uri": "(options: S3 file key or URL)",
            "filename": "(filename of file inside zip)",
            "type": "(options: [file, url])"
        }
    ]
}
```

## Testing Result
This solution has been tested with a 1.8 GB file size:

### First Attempt Setting
- File size: 1.8 GB
- Total: 3 Mins
- Duration: 100097.72 ms	Billed Duration: 100000 ms	Memory Size: 128 MB	Max Memory Used: 124 MB	Init Duration: 747.64 ms
- Result: Task timed out after 100.10 seconds

```log
START RequestId: ec9c4968-3d0d-4482-b5a3-5fb81b4d0836 Version: $LATEST
2022-03-23T08:58:47.483Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 5242880, total: undefined, part: 1, key: 'mypackage.zip' }
2022-03-23T08:58:56.041Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 10485760, total: undefined, part: 2, key: 'mypackage.zip' }
2022-03-23T08:59:03.062Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 15728640, total: undefined, part: 3, key: 'mypackage.zip' }
2022-03-23T08:59:09.601Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 20971520, total: undefined, part: 4, key: 'mypackage.zip' }
2022-03-23T08:59:16.821Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 26214400, total: undefined, part: 5, key: 'mypackage.zip' }
2022-03-23T08:59:23.404Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 31457280, total: undefined, part: 6, key: 'mypackage.zip' }
2022-03-23T08:59:29.781Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 36700160, total: undefined, part: 7, key: 'mypackage.zip' }
2022-03-23T08:59:36.381Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 41943040, total: undefined, part: 8, key: 'mypackage.zip' }
2022-03-23T08:59:42.283Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 47185920, total: undefined, part: 9, key: 'mypackage.zip' }
2022-03-23T08:59:48.561Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 52428800, total: undefined, part: 10, key: 'mypackage.zip' }
2022-03-23T08:59:54.401Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 57671680, total: undefined, part: 11, key: 'mypackage.zip' }
2022-03-23T09:00:00.561Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 62914560, total: undefined, part: 12, key: 'mypackage.zip' }
2022-03-23T09:00:06.842Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 68157440, total: undefined, part: 13, key: 'mypackage.zip' }
2022-03-23T09:00:12.582Z	ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	INFO	{ loaded: 73400320, total: undefined, part: 14, key: 'mypackage.zip' }
END RequestId: ec9c4968-3d0d-4482-b5a3-5fb81b4d0836
REPORT RequestId: ec9c4968-3d0d-4482-b5a3-5fb81b4d0836	Duration: 100097.72 ms	Billed Duration: 100000 ms	Memory Size: 128 MB	Max Memory Used: 124 MB	Init Duration: 747.64 ms	
2022-03-23T09:00:16.982Z ec9c4968-3d0d-4482-b5a3-5fb81b4d0836 Task timed out after 100.10 seconds

```


### Second Attempt with
- File size: 1.8 GB
- Total: 3 Mins
- Duration: 210068.61 ms	Billed Duration: 210069 ms	Memory Size: 512 MB	Max Memory Used: 135 MB	Init Duration: 760.79 ms
- Result: Success - Zip uploaded
```
  ------ Some logs line items are removed -----

2022-03-23T09:06:35.690Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	{  loaded: 1829765120,  total: undefined,  part: 349,  key: 'mypackage.zip'}
2022-03-23T09:06:36.334Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	{  loaded: 1835008000,  total: undefined,  part: 350,  key: 'mypackage.zip'}
2022-03-23T09:06:36.970Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	{  loaded: 1840250880,  total: undefined,  part: 351,  key: 'mypackage.zip'}
2022-03-23T09:06:37.529Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	{  loaded: 1845493760,  total: undefined,  part: 352,  key: 'mypackage.zip'}
2022-03-23T09:06:38.159Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	{  loaded: 1850736640,  total: undefined,  part: 353,  key: 'mypackage.zip'}
2022-03-23T09:06:38.670Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	{  loaded: 1855979520,  total: undefined,  part: 354,  key: 'mypackage.zip'}
2022-03-23T09:06:38.972Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	{  loaded: 1858906803,  total: 1858906803,  part: 355,  key: 'mypackage.zip'}

2022-03-23T09:06:39.259Z	6076c50c-c137-46d1-9066-b62de772271b	INFO	Zip uploaded
END RequestId: 6076c50c-c137-46d1-9066-b62de772271b
REPORT RequestId: 6076c50c-c137-46d1-9066-b62de772271b	Duration: 210068.61 ms	Billed Duration: 210069 ms	Memory Size: 1024 MB	Max Memory Used: 135 MB	Init Duration: 760.79 ms


```
