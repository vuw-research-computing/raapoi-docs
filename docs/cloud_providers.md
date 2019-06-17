
## AARNET Cloudstor

All VUW researchers have access to the AARNET (Australia’s Academic and Research
Network) Cloudstor service which provides __1 TB__ of space to each researcher.  To
use this service first login and download an appropriate client to your laptop
or desktop (or smarthone if you wish):

[Cloudstor Login](https://apac01.safelinks.protection.outlook.com/?url=https%3A%2F%2Fcloudstor.aarnet.edu.au%2Fsimplesaml%2Fmodule.php%2Fdiscopower%2Fdisco.php%3FentityID%3Dhttps%253A%252F%252Fcloudstor.aarnet.edu.au%252Fsimplesaml%252Fmodule.php%252Fsaml%252Fsp%252Fmetadata.php%252Fdefault-sp%26return%3Dhttps%253A%252F%252Fcloudstor.aarnet.edu.au%252Fsimplesaml%252Fmodule.php%252Fsaml%252Fsp%252Fdiscoresp.php%253FAuthID%253D_d8e2f7cbdf882a033df5b33385b5936c08e0e133a1%25253Ahttps%25253A%25252F%25252Fcloudstor.aarnet.edu.au%25252Fsimplesaml%25252Fmodule.php%25252Fcore%25252Fas_login.php%25253FAuthId%25253Ddefault-sp%252526ReturnTo%25253Dhttp%2525253A%2525252F%2525252Fcloudstor.aarnet.edu.au%2525252Fplus%2525252F%26returnIDParam%3Didpentityid&data=02%7C01%7Cwes.harrell%40vuw.ac.nz%7Cefa10aeb0f1d42a6cd2408d6953c9496%7Ccfe63e236951427e8683bb84dcf1d20c%7C0%7C0%7C636860484751891881&sdata=nkN7OI6g%2BKs%2FLoMG6tOtoWqk1%2FU1gbxZK%2FjCd8NNZ6c%3D&reserved=0)

NOTE: Within the Cloudstor web login settings you will need to create a Cloudstor Password, this is the password you will use to login on raapoi, it does not use your VUW credentials for the command line login.

Once you have setup your cloudstor (aka ownCloud) credentials you can use them
to sync data to and from Raapoi.  For example, if I wanted to sync my project
space to Cloudstor I would do the following from raapoi login node:

```
  tmux
  cd /nfs/scratch/harrelwe
  owncloudcmd -u wes.harrell@vuw.ac.nz project1 https://cloudstor.aarnet.edu.au/plus/remote.php/webdav/
```

The above sequence starts a tmux session to allow the transfer to continue even if I disconnect from the cluster
(type `tmux attach` to reconnect), I then change directory (_cd_) to my scratch space and then use the command `owncloudcmd` to sync my project directory called project1 to the Cloudstor service.  You will be prompted for your Cloudstor password, you can also embed this within the command line using the -p paramter.  

## Amazon AWS

A feature-rich CLI is available in raapoi.  To use it you need to load the appropriate module and its module dependencies:

  `module load amazon/aws/cli`

Before you proceed you will need to configure your environment with your Access Key ID and Secret Access Key, both of which will be sent to you once your account is created or linked.  The command to configure your environment is `aws configure`  You only need to do this once, unless of course you use more than one user/Access Key.  Most users can simply click through the region and profile questions (using the default of "none").  If you do have a specific region this should be relayed along with your access and secret keys.

Once you have the appropriate environment in place and your configuration setup you can use the aws command, plus an appropriate sub-command (s3, emr, rds, dynamodb, etc) and supporting arguments. 

More information on the CLI can be found here:
[http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-using.html](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-using.html)

#### Transferring Data to/from Amazon (AWS) S3

To transfer data from S3 you first need to setup your AWS connect, instructions for that can be found above.
Once that is done you should be able to use the aws commands to copy data to and from your S3 storage.  

For example if I wanted to copy data from my S3 storage to my project directory I could do the following:
```
  tmux
  module load amazon/aws/cli
  cd /nfs/scratch/harrelwe/project
  aws s3 cp s3://mybucket/mydata.dat mydata.dat
```
To copy something to storage simply reverse the file paths, eg.
```
  aws s3 cp mydata.dat s3://mybucket/mydata.dat 
```
The above starts a tmux session to allow the transfer to continue even if I
disconnect from the cluster (type `tmux attach` to reconnect).  I then load the
modules necessary to use the AWS commands.  I change directory to my project
space and use the aws s3 cp command to copy from S3.  More information on using
aws can be found here:
[http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#cli-aws-s3](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#cli-aws-s3)

#### Working with AWS Data Analysis Tools

Amazon has a number of data analytics and database services available.  Using the command line utilities available in raapoi, researchers can perform work on the eo cluster and transfer data to AWS to perform further analysis with tools such as MapReduce (aka Hadoop), RedShift or Quicksight.

A listing of available services and documentation can be found at the following:
[https://aws.amazon.com/products/analytics/](https://aws.amazon.com/products/analytics/)

## Google Cloud (gcloud) Connections 

The Google Cloud SDK is available in raapoi.  This includes a command-line interface for connecting to gloud services.  To get started, first load the environment module.  You can find the path with the `module spider` command.  As of this writing the current version can be loaded thusly:

  module load google/cloud/sdk/212.0.0

This will give you access to the `gcloud` command.  To setup a connection to your gcloud account use the init sub-command, eg.

  `gcloud init --console-only`

Follow the instructions to authorize your gcloud account.  Once on the Google website, you will be given an authorization code which you will copy/paste back into the raapoi terminal window.

#### Transferring Data to/from Google Cloud (gcloud)

To transfer data from gcloud storage you first need to setup your gcloud credentials, instructions for that can be found above.  Once that is done you should be able to use the `gsutil` command to copy data to and from your gcloud storage.  

For example, if I wanted to copy data from gcloud to my project directory I could do the following:
```
  tmux
  module load google/cloud/sdk/212.0.0
  cd /nfs/scratch/harrelwe/project
  gsutil cp gs://mybucket/mydata.dat mydata.dat
```
To copy something to storage simply reverse the file paths, eg.
```
  gsutil cp mydata.dat gs://mybucket/mydata.dat 
```

The above starts a tmux session to allow the transfer to continue even if I
disconnect from the cluster (type `tmux attach` to reconnect).  I then load the
modules necessary to use the gsutil commands.  I change directory to my project
space and use the gsutil cp command to copy from gcloud.  More information on
using gcloud can be found here:
[https://cloud.google.com/sdk/gcloud/](https://cloud.google.com/sdk/gcloud/)

#### Working with GCloud Data Analysis Tools

Google Cloud has a number of data analytics and database services available.  Using the gcloud command line utilities available on raapoi, researchers can perform work on the cluster and transfer data to gcloud to perform further analysis with tools such as Dataproc (Hadoop/Spark), BigQuery or Datalab (Visualization)

A listing of available services and documentation can be found at the following:
[https://cloud.google.com/products/](https://cloud.google.com/products/)

## DropBox Cloud Storage

__NOTE:__ Dropbox has upload/download limitations and we have found that once your file gets above 50GB in size the transfer will have a better chance of timing out and failing.

Configuring your Dropbox account on raapoi

__Step A:__  On your local laptop or desktop start your browser and login to your Dropbox account

__Step B:__ On Raapoi type the following:

   `module load dropbox`

__Step C:__ Setup account credentials (You should only need to do this once):

Run the following command from Raapoi

   `dbxcli account`

You will now see something like the following:

  1. Go to https://www.dropbox.com/1/oauth2/authorize?client_id=X12345678&response_type=code&state=state
  2. Click "Allow" (you might have to log in first).
  3. Copy the authorization code.
  Enter the authorization code here:

__Step D:__

Copy the URL link listed in Step C1 and paste it into the web browser that you started in Step A

This will provide you with a long access code (aka hash).  Now copy that access code and paste it into your Raapoi terminal after Step C3 where it is asking for Enter the authorization code here 

Now hit enter or return.  You should see that you are now logged in with your Dropbox credentials

#### Basic Dropbox commands

Remember to load the dropbox environment module if you have not already (see `module spider` for the path)

Now type `dbx` or `dbxcli` at a prompt.  You will see a number of sub-commands, for instance ls, which will list the contents of your Dropbox, eg

  `dbxcli ls`

#### Downloading from Dropbox

Downloading uses the subcommand called: get.   The basic format for get is:

  `dbxcli get fileOnDropbox fileOnRaapoi`

For instance, if I have a datafile called 2018-financials.csv on Dropbox that I want to copy to my project folder I would type:

  `dbxcli get 2018-financials.csv /nfs/scratch/harrelwe/projects/finance_proj/2018-financials.csv`

#### Uploading to Dropbox

Uploading is similar to downloading except now we use the subcommand: put.  The basic format for put is:

  `dbxcli put fileOnRaapoi fileOnDropbox`

For example I want to upload a PDF I generated from one of my jobs called final-report.pdf I would type:

  `dbxcli put final-report.pdf final-report.pdf`

This will upload the PDF and name it the same thing, if I wanted to change the name on Dropbox I could:

  `dbxcli put final-report.pdf analytics-class-final-report.pdf`

