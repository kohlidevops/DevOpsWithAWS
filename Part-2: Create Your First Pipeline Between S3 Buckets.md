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






