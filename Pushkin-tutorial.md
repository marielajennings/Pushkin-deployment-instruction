## Requirements
You need git for the various steps of deployment. If you are working on a Mac, you have git already installed. If you are working on a PC, make sure you install git and familiarize yourself with the basic commands before proceeding.

You will need a [GitHub](https://github.com) account. Ensure that you have created one before proceeding.

You will need to install [Docker](https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac) and [create an account] (https://id.docker.com).

You will need a Postgres manager. For example, you can download [SQLPro for Postgres](https://macpostgresclient.com/).

You will need an [Amazon Web Services](https://aws.amazon.com/free/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=cloud_computing_b&sc_content=aws_url_e_control_q32016&sc_detail=amazon.%20web%20services&sc_category=cloud_computing&sc_segment=188908164670&sc_matchtype=e&sc_country=US&s_kwcid=AL!4422!3!188908164670!e!!g!!amazon.%20web%20services&ef_id=WUGhAAAAAHs2P1qP:20171016145411:s.) account.

Some repository and file names should be changed when customizing the setup. However, the instructions will contain the original names for simplicity. Make sure you replace all the relevant names in any commands you execute.

##Forking the Relevant Repos to Work on a Custom Setup
Once you have logged into your GitHub account, navigate to the [Games With Words organization](https://github.com/gameswithwords). Click on [**gww**](https://github.com/gameswithwords/gww) and fork the repository. Change the name of the repository (Settings->Repository name). 
Return to the [Games With Words organization](https://github.com/gameswithwords), and fork [**cron**](https://github.com/gameswithwords/cron), [**experiments**](https://github.com/gameswithwords/experiments), and [**front-end**](https://github.com/gameswithwords/front-end).

##Local Deployment and Testing

###Setting Up Databases

Open your Postgres manager, select **New** and fill out the following fields:

**Server host:** localhost  
**Login:** postgres  
**Server port:** 5432  
**Database:** dev

Create another database with the following information:

**Server host:** localhost  
**Login:** postgres  
**Server port:** 5433  
**Database:** transactions_dev

### Front End Deployment
  
* Open Terminal
* You will need to clone the forked and renamed **gww** repository:   

```
 git clone [URL]   
```

<span style="color:red">**Important Note**:</span> When cloning, always use https instead of ssh.

* Change the working directory to the folder containing the cloned **gww** repository.
* To place the contents of the **cron**,**experiments**, and **front-end** repositories in the appropriate submodules of **gww**, you will need to remove the submodules which got cloned along with the **gww** repository. The commands to do that are as follows:   

```
$ git submodule deinit cron
  
$ git rm –f cron

$ git submodule deinit experiments

$ git rm -f experiments

$ git submodule deinint

$ git submodule deinit front-end

$ git rm -f front-end
```

* Next, you will need to add the deleted submodules from the forked repositories on your GitHub account (again, these are **experiments**, **cron**, and **front-end**). Replace [URL] in the following command with the GitHub URL for each of those repositories before executing:

```
$ git submodule add [URL] 
```

* After adding front-end, the submodule jsPsych will remain empty. To add the contents of jsPsych, go back to the front-end repository on GitHub, find the jsPsych submodule and get its URL. From the main **gww** folder execute the following commands to add the jsPsych submodule:

```
$ git submodule deinit jsPsych

$ git rm jsPsych

$ git submodule add [URL]
```
* Navigate to the [pushkin-npm organization](https://github.com/pushkin-npm) on GitHub. Click on [pushkin-cli](https://github.com/pushkin-npm/pushkin-cli).
* Clone this repository in the directory containing **gww** (`git clone [URL]`).
* Change your working directory to the folder containing the cloned **pushkin-cli** repository and install:

```
$ cd pushkin-cli

$ npm install –g ./
```
To test whether the Pushkin installation was successful run the command `pushkin`. If the installation was successful, you will get a list of commands that you can execute:

![logo](https://github.com/marielajennings/Tutorial-images/raw/master/Screen%20Shot%202017-08-25%20at%203.29.12%20PM.png)


* Change the working directory back to **gww** (`cd gww`) and run `pushkin sync`

* Change the working directory to **front-end** (`cd front-end`) and run the following commands:

```
$ npm install

$ npm start
```
At this point you should be able to see the front end of the website in your browser.

### Setting Up a Running Docker Container
In **gww**:
//Provided that you have installed Docker and created a Dockerhub account, you should be able to start a Docker container by running the following command:

```
$ docker-compose -f docker-compose.debug.yml up
```
After the Docker container has been initiated, `cd` into `db-worker` and run `docker ps`. This command will give you a list of containers with their IDs, STATUS, PORTS, and NAMES. Copy the container ID of `db-worker`

![logo](https://github.com/marielajennings/Tutorial-images/raw/master/Screen%20Shot%202017-08-31%20at%2012.27.35%20PM.png)

Run the following command:

```
$ docker exec -it CONTAINER_ID bash
```

Followed by:

```
$ npm run migrations
```
If you have a stimuli file that you would like to use for a quiz, you can run the following command:

```
$ node seeder.js NAME_OF_QUIZ
```
A series of questions will appear; the answer to all of them should be **Yes**.

![logo](https://github.com/marielajennings/Tutorial-images/raw/master/Screen%20Shot%202017-08-31%20at%2012.43.22%20PM.png)

To sync all of the updates, run `pushkin sync` from the **gww** folder.

***IMPORTANT NOTE:*** Every time you make changes to files you will need to rebuild the Docker containers, since the changes do not get updated automatically. To do this, run the following command:

```
$ docker-compose -f docker-compose.debug.yml up --build
```

### Creating a New Quiz

#### The Back End


***IMPORTANT NOTE:*** Do **NOT** use uppercase letters in the name of the new quiz. This will cause problems when the Docker containers are rebuilt after the quiz has been added.

To begin, you will need to `cd` into the **gww** folder. Pushkin offers the option of generating the basic files needed for a new quiz. Alternatively, you can modify the files for an existing quiz. To generate the template files for a new quiz run:

```
$ pushkin generate dbItems QUIZ_NAME
```
OR

```
$ pushkin scaffold QUIZ_NAME
```

All the scaffolded files for a quiz can be found in a folder with the quiz name located in the **experiments** folder. After those files have been modified to reflect the requirements of the new quiz, run the following commands:

```
$ pushkin sync
```
After syncing, open the **docker-compose.debug.yml** file and make sure that the new quiz has been added to the end of the file. Pay attention to the **image**. Make sure it follows the same format as previous quizzes. When all the required changes have been made, run:

```
$ docker-compose -f docker-compose.debug.yml up --build
```

to rebuild the Docker containers, followed by:

```
$ cd db-worker
$ docker ps
```
Select the Container ID of db-worker and run:

```
$ docker exec -it CONTAINER_ID bash
$ npm run migrations
$ node seeder.js NAME_OF_QUIZ
```
A series of questions will appear; the answer to all of them should be **Yes**.

#### The Front End

To create the front end for the quiz,

In the `/front-end/experiments` folder, each quiz has its own folder with the React code for the quiz and the jsPsych plugins used by the quiz. You will need to create a new folder with those files for the quiz you want to publish. 
Additionally, you will need to add the link to the quiz in `front-end/pages/quizzes/index.js`.
The last step is adding your new quiz to `/front-end/core/routes.js'



and add the files for the new quiz or copy and update the files from an existing quiz to reflect the content of the new quiz. The stimuli for the quiz can be added into the `public\quizzes` folder located inside the `front-end` folder.

You will also need to add the quiz in the `pages\quizzes\index.js`.

##Live Deployment

Once you have confirmed that your website works locally, you can begin live deployment. At this point, you should have a working Dockerhub ID. Ensure that you are signed in to Dockerhub on your computer before proceeding.

#### Amazon Web Services (AWS)
Open your AWS account, and go to **Storage** -> **S3**. Create a bucket with the following configuration:

(add settings for gww bucket)
Allow public read access

Once the bucket is created, click on it and select **Permissions** -> **Bucket Policy**. Use the following policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForGetBucketObjects",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::BUCKET_NAME/*"
        }
    ]
}

```

changing bucket BUCKET_NAME to the name you gave your S3 bucket. You will also need to change the CORS configuration to:

```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <MaxAgeSeconds>3000</MaxAgeSeconds>
    <AllowedHeader>Authorization</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

After saving the S3 bucket settings, open the file **run.js** located in the **front-end** folder. On line 112, change the bucket name to the name of the bucket you just created:

```
 s3Params: { Bucket: 'YOUR_BUCKET_NAME' },
```

Go back to **AWS** -> **Cloudfront** -> **Create Distribution** -> **Web**.
Use the following settings:

* Origin Domain Name: YOUR_BUCKET.s3.amazonaws.com
* Origin ID: S3-YOUR_BUCKET
* Viewer Protocol Policy: HTTP and HTTPS
and **Create Distribution**.

Click on the newly created distribution and go to **Behaviors** -> **Edit** -> **Cache Based on Selected Request Headers** -> **Whitelist**
From the Filter Headers options select **Origin** -> **Add** and then select **Yes, Edit** on the bottom of the settings page to finish.

From the **Cloudfront Distributions** page take the domain name of your distribution and insert it on line 22 of **run.js**:

```
url: 'https://YOUR_DISTRIBUTION.cloudfront.net/',
```

#### Building and tagging containers

In a new Terminal window, `cd` into `front-end` and run the following script (editing the relevant parts first!):

```
npm run publish &&
cd .. &&
cp -rf ./front-end/public/**.ico ./server/html &&
cp -rf ./front-end/public/**.txt ./server/html &&
cp -rf ./front-end/public/**.html ./server/html &&
cp -rf ./front-end/public/**.xml ./server/html &&
docker build -t DOCKERHUB_ID/api:latest api &&
docker build -t DOCKERHUB_ID/cron:latest cron &&
docker build -t DOCKERHUB_ID/db-worker:latest db-worker &&
docker build -t DOCKERHUB_ID/server:latest server &&
docker build -t DOCKERHUB_ID/QUIZ_NAME:latest workers/QUIZ_NAME &&
docker push DOCKERHUB_ID/api &&
docker push DOCKERHUB_ID/cron &&
docker push DOCKERHUB_ID/db-worker &&
docker push DOCKERHUB_ID/server
docker push DOCKERHUB_ID/QUIZ_NAME 

```

#### Security Groups 

(Add info on how to make them here)

#### Creating Databases

Go back to **AWS** -> **Services** -> **RDS** -> **Launch DB Instance**

1. **Amazon Aurora** (for Rancher)

* Instance specifications:

	DB instance class: db.t2.small        
	Multi-AZ deployment: No   
	DB instance identifier: YOUR\_IDENTIFIER       
	Master Username: YOUR\_USERNAME (will be used to log in to the database)     
	Master Password: YOUR_PASSWORD   
	VPC Security groups: Select existing VPC security groups: YOUR\_SECURITY\_GROUP     
	DB Cluster Identifier: YOUR\_IDENTIFIER  
	Database name: YOUR\_DATABASE\_NAME
	Database port: 3306   
	Backup retention period: 7 days
	Maintenance window: Select window: SELECT\_A\_WINDOW    
	
	At this point you are ready to click on **Launch Instance** and move on to creating the other databases.
	
2. **PostgreSQL** (main database)

* Instance specifications:

	DB instance class: db.t2.medium        
	Multi-AZ deployment: Yes     
	Allocated storage: 100GB  
	DB instance identifier: YOUR\_IDENTIFIER       
	Master Username: YOUR\_USERNAME (will be used to log in to the database)     
	Master Password: YOUR_PASSWORD   
	VPC Security groups: Select existing VPC security groups: YOUR\_SECURITY\_GROUP       
	Database name: YOUR\_DATABASE\_NAME   
	Database port: 5432   
	Backup retention period: 7 days
	Maintenance window: Select window: SELECT\_A\_WINDOW    
	
3. **PostgreSQL** (transactions database)

* Instance specifications:

	DB instance class: db.t2.small       
	Multi-AZ deployment: Yes     
	Allocated storage: 20GB  
	DB instance identifier: YOUR\_IDENTIFIER       
	Master Username: YOUR\_USERNAME (will be used to log in to the database)     
	Master Password: YOUR_PASSWORD   
	VPC Security groups: Select existing VPC security groups: YOUR\_SECURITY\_GROUP       
	Database name: YOUR\_DATABASE\_NAME   
	Database port: 5432   
	Backup retention period: 7 days
	Maintenance window: Select window: SELECT\_A\_WINDOW 
	
When you are done creating the databases, go back to the list of **Instances** and get the endpoints for the databases you created by clicking on them and scrolling down to **Connect** -> **Endpoint**

#### Configuring ***docker-compose.production.yml***

Go back to the main repository folder for the project, and find the file called **docker-compose.production.yml**. Open the file in a text editor and change the following:

* For every image, change the DOCKERHUB_ID to your Dockerhub ID;
* Insert the DATABASE\_URL and TRANSACTION\_DATABASE\_URL (the format for those is: **postgres://user_name:password@host:port/database_name**), they appear twice - under environment for **cron** and for **db-worker**).

### Creating a Rancher Instance on AWS

Go back to your **AWS account** -> **Launch Instance** -> **AWS Marketplace** -> **Search** -> **Rancher** and select the first option.

* Instance configuration:    
  Type: t2.medium  
  Enable termination protection: Yes    
  Size: 16GiB  
  Add Tag: key=Name, value=NAME\_OF\_RANCHER\_INSTANCE   
  Security Group: Select Existing: RANCHER\_SECURITY\_GROUP
  Key Pair: your choice, but make sure you have access to the private key for the pair you choose! 
 
 Launch the instance!
 
 #### Elastic IP
 
 In your AWS account, find **Elastic IP** (you can use the search field) -> **Allocate new address** -> **Allocate**
 Once it has been created, select it and from the **Actions** menu select **Associate address**. Select your Rancher instance and then **Associate**
 
### Setting Up the Rancher Instance
I recommend using an SSH client such as [Termius](https://www.termius.com/) for this step, but you can also SSH to your instance from Terminal on your Mac (`ssh –i path_to_key  rancher@IP_OF_INSTANCE `). 

**NOTE**: The IP of the Rancher instance will be the Elastic IP you just associated with it.

Create a directory named rancher (`mkdir rancher`) and `cd` into it.

Go back to your AWS account. Your domain name should appear under **Hosted Zones**. Click on it and then on **Create Record Set**


        


