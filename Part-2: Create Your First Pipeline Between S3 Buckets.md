# To create a AWS CodePipeline between S3 buckets

## Create a Source bucket

To create a source bucket with versioning enabled - This is because CodePipeline identifies the source revisions from their version id.


![image](https://github.com/user-attachments/assets/6ad6541b-2009-4c57-9949-36e01f23f2dd)


## Create a Production bucket

To create a production bucket with public access enabled - This is because anyone can access it from anywhere as it is website hosting


![image](https://github.com/user-attachments/assets/d3a672c8-5aea-4988-8591-a58a6ab3045c)


To make this bucket host static website. First we will add a bucket policy - Bucket > Permission > Edit > Bucket Policy

You can find the bucket-policy.json - https://github.com/kohlidevops/DevOpsWithAWS/blob/main/bucket-policy.json


![image](https://github.com/user-attachments/assets/9b892cd4-bc1c-40b0-bddd-5a89ed22bcd8)


To make this bucket static website hosting - Properties > Static Website hosting > Enable > Hosting type > Host a static website

```
index document - index.html
error document - error.html
```

To find the zip file which contains index and error.html files - https://github.com/kohlidevops/DevOpsWithAWS/blob/main/my-website.zip

To upload the my-website.zip to source bucket


![image](https://github.com/user-attachments/assets/32a8ab42-6090-4a3b-be95-0047e4dfe704)


## To create a CodePipeline

To create a codepipeline with build custom pipeline


![image](https://github.com/user-attachments/assets/be1e31c9-1ce5-44b4-9bbb-8f4bd784d453)


Give a Pipeline name and allow to create a new service role


![image](https://github.com/user-attachments/assets/6188a4a9-1fde-4969-9433-b417deb7177c)


To add a source stage as S3 bucket and place the object key

To skip Build and Test stage as we are not using

To select the Deploy stage > Deploy provide should be S3 prod bucket and file should extract before deploy


![image](https://github.com/user-attachments/assets/4059c332-b969-43dd-8556-06961c6104d4)


Review and create a Pipeline > The Pipeline should trigger once it is created.


![image](https://github.com/user-attachments/assets/b009fb04-f7f2-4ae4-9bda-8afbcacb379b)


The files are extracted and deployed into prod buckets


![image](https://github.com/user-attachments/assets/cf5ac6dc-afaf-43c9-b571-43f67ebe4ce5)


Then Why not access the S3 Static website hosting endpoint?


![image](https://github.com/user-attachments/assets/a78caa1c-ca17-44f7-879e-5cb919aa60cf)







