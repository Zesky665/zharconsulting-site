
### Client
```
boto3.client("s3")
```

### Resource
```
boto3.resource("s3")
```

### S3 Permissions

### S3 path
The Path for a S3 object is comprised to two parts, the bucket name and the prefix. 
So an object with the path of `test_bucket/test_folder/test.csv` will have a `bucket_name` of `test_bucket` and a prefix of `/test_folder/test.csv`.
The prefix can does not have to be the exact location of the object, something like `/test_folder/` would match every file in that folder.  

### Listing Objects

```
def listFiles(bucket, prefix, client):

response = client.list_objects(Bucket=bucket, Prefix=prefix)

# if no files found return empty list
if "Contents" not in response:

	return []

# Filter out only the file names
keys = [k["Key"] for k in response["Contents"] if not k["Key"].endswith("/")]

keys.sort()

return keys
```
### Downloading Objects

```
client.download_file(bucket, file, file_path)
```
### Uploading Objects

```
client.upload_file(filename, bucket, key)
```

### Reading Objects

```
response = client.get_object(Bucket='your-bucket-name', Key='your-object-key')
object_content = response['Body'].read().decode('utf-8')
```

### Writing Objects

Writing text directly to a file, this will create a file with the given name.
```
client.Object("my-bucket", "object_name.txt").put( Body="Hello, World!" )
```

Writing json to a file.
```
data = {"key": "value"} 
json_str = json.dumps(data) 
client.put_object( Bucket="my-bucket", Key="object_name.json", Body=json_str, )
```
### Deleting Objects

```
client.delete_object(
    Bucket='my-bucket',
    Key='example/file.txt'
)
```