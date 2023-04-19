## 1. s3上传目录
```
# s3 = boto3.client('s3', region_name=region, aws_access_key_id=aws_access_key_id, aws_secret_access_key=aws_secret_access_key)
def upload_s3_dataset(s3, bucket, local_dir, dst_s3_path):
  try:
    stream = tqdm(os.walk(local_dir))
    for path, subdirs, files in stream:
      for file in files:
        extra = {}
        file_type = os.path.splitext(file)[-1].lower()
        if file_type == ".jpg":
          extra["ContentType"] = "image/jpg"
        elif file_type == ".jpeg":
          extra["ContentType"] = "image/jpeg"
        elif file_type == ".png":
          extra["ContentType"] = "image/png"

        dst_path = path.replace(local_dir, "").replace(os.sep, '/')
        local_file = os.path.join(path, file)
        s3_file = '{}/{}/{}'.format(dst_s3_path, dst_path, file).replace('//', '/')
        
        s3.upload_file(local_file, bucket, s3_file, extra)
        
        stream.set_description('Uploaded {} to {}'.format(local_file, s3_file))
  except Exception as e:
    print('Upload data failed. | src: {} | dst: {} | Exception: {}'.format(local_dir, dst_s3_path, e))
    return False

  print("\n\n")
  print("Successfully uploaded {} to S3 {}".format(local_dir, dst_s3_path))
  return True
```

## 2. s3下载目录
```
# s3 = boto3.resource('s3', region_name=region, aws_access_key_id=aws_access_key_id, aws_secret_access_key=aws_secret_access_key)
def download_s3_folder(s3, bucket_name, s3_folder, local_dir):
  """
  Download the contents of a folder directory
  Args:
    bucket_name: the name of the s3 bucket
    s3_folder: the folder path in the s3 bucket
    local_dir: a relative or absolute directory path in the local file system
  """
  bucket = s3.Bucket(bucket_name)
  stream = tqdm(bucket.objects.filter(Prefix=s3_folder)) 
  for obj in stream:
    target = obj.key if local_dir is None else os.path.join(local_dir, os.path.relpath(obj.key, s3_folder))
    if not os.path.exists(os.path.dirname(target)):
        os.makedirs(os.path.dirname(target))
    if obj.key[-1] == '/':
        continue
    bucket.download_file(obj.key, target)
    stream.set_description('Downloaded {} to {}'.format(obj.key, target))
```
