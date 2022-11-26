1. Clone the repo, and checkout to a new branch, (say) dev. 

2. Create a .gitignore file, and add starter/terraform/.terraform. The binaries created by terraform are too large to be
pushed to a remote repo. 

###### PROJECT BEGINNING ######

3. Create two S3 buckets in us-east-1. One will contain the AMI Image in us-east-1. The second bucket is the one terraform will use.
``` aws ec2 create-restore-image-task --object-key ami-08dff635fabae32e7.bin --bucket udacity-srend --name "udacity-<ami_bucket_name>" ```

4. The command above will send back an AMI image. Make note of this ID.

5. Use ``` aws ec2 copy-image --source-image-id <your-ami-id-from-above> --source-region us-east-1 --region us-east-2 --name "udacity-<your_name>" ```
to create a copy of the above ID in us-east02.

6. Go to us-east-2 in the console, and create a private key pair named udacity. This is required to create and ssh into the EC2 instance. 

7. Open the _config.tf and change the bucket name and region to the s3 bucket that you have created in a previous step.

8. Further replaced the /starter/terraform/modules/ec2/ec2.tf to the following file: 

