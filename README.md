# fMRIPrep-AWS
How to set up an up-to-date fMRIPrep pipeline on AWS

# Prologue
Thanks to Mingwen Dong and Mengxue Kang for their 2019 AWS Open Blog post on running fMRIPrep on AWS: [fMRI data preprocessing on AWS using fMRIprep](https://aws.amazon.com/blogs/opensource/fmri-data-preprocessing-aws-fmriprep/).

This GitHub page builds upon their work, providing a step-by-step guide to setting up the latest fMRIPrep pipeline on AWS. Since the original AWS blog post was published, fMRIPrep has been updated to version 24.x (24.1.1 at the time of writing), incorporating many new features and improvements. Setting up the latest pipeline will allow you to use these enhancements in your analysis. 

# Walkthrough
The following section covers the procedures to set up an up-to-date FMRIPREP pipeline on AWS:
1. [Create an EC2 instance](https://github.com/jwu103/FMRIPREP-AWS/tree/main?tab=readme-ov-file#create-an-ec2-instance)
2. [Create an Amazon Machine Image from the EC2 instance](https://github.com/jwu103/FMRIPREP-AWS/tree/main?tab=readme-ov-file#create-an-amazon-machine-image-from-the-ec2-instance)
3. [Update the CloudFormation setting](https://github.com/jwu103/FMRIPREP-AWS/tree/main?tab=readme-ov-file#update-the-cloudformation-setting)
4. [Clean up and voilà](https://github.com/jwu103/FMRIPREP-AWS/tree/main?tab=readme-ov-file#clean-up-and-voil%C3%A0)

## Steps
### Create an EC2 instance
1. Enter "EC2" into the AWS search bar and launch EC2 service.<br>
![EC2_launch](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_launch.jpg)

2. Click "Launch instances" as shown on the top right corner.<br>
![EC2_launch_instances](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_launch_instance.png)

3. Enter a name for the EC2 instance; I am using "fMRIPrep_base" as the name here.<br>
![EC2_instance_name](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_name.jpg)

4. Select Ubuntu as the operating system for the EC2 instance.<br>
![EC2_instance_OS](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_os.jpg)

5. Select an instance type for the EC2 instance. There are two factors that go into this decision: need and pricing. For fMRIPrep, the developers provide benchmarking data on the [website](https://fmriprep.org/en/stable/faq.html#how-much-cpu-time-and-ram-should-i-allocate-for-a-typical-fmriprep-run). I typically allocate 32 GB of ram and 8 CPUs when running fMRIPrep. For this illustration, I picked m6a.2xlarge as the instance type.<br>
![EC2_instance_type](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_type.jpg)

6. Select a Key pair. If you didn't have one, click "Create new key pair".<br>
![EC2_instance_keypair](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_keypair.jpg)

7. Configure the storage volume for the EC2 instance. I am using 30 GB in the illustration, but it's possible to use less.<br>
![EC2_instance_storage](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_storage.jpg)

8. Launch the instance. You should see a message as shown below. Then, click "Connect to instance".<br>
![EC2_instance_success](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_success.jpg)

9. Click "Connect" at the bottom right corner.<br>
![EC2_instance_connect](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_connect.jpg)

10. Afer connecting to the instance, there are three packages to install: [Docker](https://docs.docker.com/engine/install/ubuntu/), unzip, and [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html). For ease, I've pasted the codes below. Copy the codes, and paste them into the instance, and then hit enter. When pasting, right click on the instance and select "paste".<br>

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Wait for the installations to complete, and you should see the message as shown below.<br>
![EC2_instance_install](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_install.jpg)

#### *Note: As you may notice, I did not include a step to install fMRIPrep directly into this EC2 instance, which will be used to create the Amazon Machine Image. This is because I generally prefer to call the Docker package in the ```docker run``` command, and when Docker can't find the package, it will download it. If you want to pre-install fMRIPrep here, please make sure to reference the correct version number in the ```docker run``` command in the Lambda function.*

### Create an Amazon Machine Image from the EC2 instance
1. Go back to the EC2 dashboard page. Select the EC2 instance. Open the "Action" tab. Go to "Image and templates". Then, click "Create image". This will create an Amazon Machine Image using the EC2 instance.<br>
![EC2_instance_createimage](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_createimage.jpg)

2. Go through the process and you should see a message as shown below.<br>
![AMI_creating](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/AMI_creating.jpg)

3. The process of creating the Amazon Machine Image can take a bit of time. You can navigate to the AMIs page under the Images tab on the left sidebar. If the *Status* of the AMI says "Pending", that means the Image is still being created. The process will be completed when the Status changes to "available".<br>
![AMI_available](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/AMI_available.jpg)

4. After creating the AMI, don't forget to terminate the EC2 instance! Otherwise you will continue to be charged for it!<br>
![EC2_instance_terminate](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/EC2_instance_terminate.jpg)

#### *Note: The AMI created from this illustration is kept. If you prefer not to create your own, you can use this AMI by referencing its ID: ```ami-0cb3e694642e383aa```.*

### Update the CloudFormation setting
At the time of writing this post, Python runtime 3.7 no longer seems to work for the Lambda function. Therefore, in order to set up all of the resources to run fMRIPrep on AWS as illustrated in the original AWS blog post, you need to either modify the Lambda function setting in Infrastructure Composer or modify the json file that the AWS blog post used.<br>

For ease, I've modified the json file and uploaded to this [GitHub page](https://github.com/jwu103/FMRIPREP-AWS/blob/main/fmriprep-cf.json), as well as an S3 bucket. You can either download the json file, create the stack in CloudFormation by uploading the json file, or [create the stack by retrieving the json file from the S3 bucket](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create?stackName=fmriprep&templateURL=https://smallstorage62.s3.us-west-2.amazonaws.com/fmriprep-cf.json). If the hyperlink above doesn't work, you can copy the S3 link ```https://smallstorage62.s3.us-west-2.amazonaws.com/fmriprep-cf.json``` and paste it into the Amazon S3 URL field when you create the stack, and click Next.<br>

Here I will explain how to modify the Lambda function setting:

1. After pasting the S3 URL into the Amazon S3 URL field in CloudFormation or uploading the template json file, click "View in Infrastructure Composer".<br>
![Infrastructure_composer](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/Infrastructure_composer.jpg)

2. Locate the FmriprepLambda component, and click "Details".<br>
![Lambda_change](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/Lambda_change.jpg)

3. A window should appear on the right. In the window, you can change the version of the Python runtime, and environmental variables such as (a) the Amazon Machine Image that the fMRIPrep pipeline will use, (b) the EC2 instance type that the pipeline will use to perform the analysis, (c) the unique user ID, (d) the name of the freesurfer license, and (e) the name of the folder in the input S3 bucket that will contain the BIDS neuroimaging data.<br> 
![python_runtime](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/python_runtime.jpg)
![Lambda_environmental_variable](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/Lambda_environmental_variable.jpg)

4. After you finish editing the Lambda function settings, save them and proceed.

### Clean up and voilà
Once you finish creating the stack using CloudFormation, you will have mostly everything you need to run the latest version of fMRIPrep on AWS! But since fMRIPrep has now moved to nipreps, you will need to modify the Lambda function to correctly call the Docker package. 

1. Locate the line that starts with "docker run" in fmriprep-lambda.py. Change ```poldracklab/fmriprep:latest``` to ```nipreps/fmriprep:latest```, or to ```nipreps/fmriprep:<version>```.
![Docker_function](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/Docker_function.jpg)

2. Click "Deploy" on the left side bar, or press Ctrl+Shift+U.

3. Upload your freesurfer license to the freesurfer license S3 bucket, and your BIDS-formatted neuroimaging data to the bids folder in the input S3 Bucket. And now you are in business! Click the "Test" button on the Lambda function page or press Ctrl+Shift+I, and fMRIPrep will start running.
![Docker_function](https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/Docker_function.jpg)

# Epilogue
1. This is essential a guide on how to build an Amazon Machine Image with Docker and AWS preloaded, such that when the Lambda function is executed, the Docker image of your choice can be downloaded and run. The steps outlined on this GitHub page can also be generalized to set up other Docker packages, such as XCP-D (for resting-state denoising), qsiprep/qsirecon (for structural preprocessing and reconstruction), and fmripost-aroma (for ICA-AROMA denoising if using fMRIPrep version 23.1.0 or later). Also, it is possible to create a single Amazon Machine Image with one or multiple packages pre-loaded; you will just need to make sure the Lambda function is calling the correct versions of the packages that you installed on the AMI.
2. I am neither a computer scientist nor an AWS expert. However, I once found myself struggling to figure out how to run analyze fMRI data for my research without access to local high-performance computing. My hope in writing this guide is that it might help someone who is facing a similar challenge some day.
3. If you find this page helpful, I will humbly accept your donation in the form of a star. ;)
4. A dog picture for the road.<br> <p align="center"><img src="https://github.com/jwu103/Picture_Reference/blob/main/Picture_Reference/Peaceful_dog.jpeg" width="300"></p>
