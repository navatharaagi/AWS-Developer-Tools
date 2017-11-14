# Manage and Deploy Code with AWS Developer Tools
### AWS CodeCommit
- It is a managed source code control service that host private Git repos
- CodeCommit is a communication tool
- It is an service that allows developers to collaborate on a project & easily manage,share,update & coordinate the code they are independently working on.
Benefits:
- Highly available,scalable & fault tolerant
- No size limit
- Integrates with other AWS services(Code-pipeline,Lambda,SNS)
- Easily migrate files from other Git-based repos
- Works with existing Git-based tools
Setup & Config:
- Tools we need to use code commit are AWS CLI(Edit,Create,Del,view repos), Git.
- Communication protocols are SSH & HTTPS
HTTPS or SSH:
- Storing & verifying credentials is a main diff
    - HTTPS: Credentials Helper
    - SSH: RSA Key Pairs  
- Port access (our firewall & network security)
    -HTTPS: Port 443
    -SSH: Port 22 [more efficient]
- Recommendation:
    - Windows & linux: HTTPS
    - MacOSX: SSH (due to keychain issues)

### Setup & Configuration
#### 1. Configure IAM Users, Access Keys & Policies
- Create a IAM User(as CodeCommit),download key credentials of that user.
- Attach AWSCodeCommitFullAccess Policy to the user.
#### 2. OSX: GIT Installation
- Download git for MacOSX
- To check whether git installed or not, run 'git help' command in CLI.
#### 3. Linux: GIT Installation
- Download git for linux or unix by following given commands according to distribution like CentOS(yum), ubuntu,…..from Google
#### 4. OSX/Linux: AWS CLI Installation
```sh
$python --version       /* to check whether python installed or not
@root$sudo yum install python3     /* to install python for CentOS
@root$sudo apt-get install python3    /* for  Debian distributions
@root$pip --help   /* to check pip installation
@root$curl -0 https://bootstrap.pypa.io/get-pip.py    /* to install pip
@root$sudo python3 get-pip.py
@root$sudo pip install awscli     /* to install AWS CLI
@root$sudo pip3 install awscli --ignore-installed six   /* if we get "error:six" while AWS installation,install this ignore command
@root$aws configure
AWS Access Key ID : Copy & paste access key of IAM user (code commit)
AWS Secret Key: Copy & paste secret key of IAM user (code commit)
region: enter region
format: json
```
Note: if we want to use another IAM user,we should give that IAM User AWS access key & secret key by running aws configure
#### 5. OSX/Linux: Configure SSH Credentials (SSH)
```sh
@root$cd ~/.ssh    /*change dir to ssh
@root$ssh-keygen    /*to generate a rsa key pair
enter file to save key: codecommitkey  /* can give any name
enter password/passphrase:            /* don’t enter anything
@root$ls         /* lists codecommitkey , codecommitkey.pub files  
@root$cat codecommitkey.pub    /*copy the key
```
- Goto AWS Console:
  - AWS—>IAM->Policies—>IAMUserSSHKeys->click on it->Attach—>select “codecommitUser”-> Attach policy
  - IAM—>Users—>codecommituser—>security credentials tab—>upload ssh public key—>
Paste the copied key of codecommitkey.pub—>upload—SSH Key ID—>copy ssh id

- Goto CLI:
```sh
@root$touch config
@root$nano config        /*to edit config file in nano editor,paste the following code
Host git-codecommit.*.amazonaws.com
            User <Input SSH key>     /*paste copied ssh id
            Identityfile ~/.ssh/<NAME of PUB FILE> /*codecommitkey is name of pub file
save & exit
@root$chmod 600 config
@root$ssh git-codecommit.us-east-1.amazonaws.com    /* Successfully authenticates
```
#### 6. OSX/Linux: Configure the Credential Helper (HTTP)
```sh
$git config --global credential.helper ‘!aws --profile codecommit credential-helper $@‘     /*  config credential-helper globally,put string inside .config file & points to aws credentials in git
$git config --global credential.UseHttpPath true   /* telling git that we are using Http as communication protocol with our repo
$git config --global --edit    /*opens vim editor file, to view git config file with given credentials
```
Note: if IAM user credentials(access,secret keys) changed in AWS CLI, we have to re-run the above git commands to reconfigure the credential helper.
- For linux the above commands are enough
- For OSX:
- credentials last for 15mins only, its issue in OSX(Mac).So git clone ‘macerrorrepo’ to our local repo.
```sh
@mac$git clone <url of macerror repo> <local-new-repo-name>
$cd local-repo
$touch test.txt
$git add text.txt
$git commit -m “test commit”
$git push
```
- After 15 mins, if we do git pull, we will get error.To avoid this
- Goto “keychain Access” in our Mac sys,search for ‘git’, it will show git keys, select the key which associated with macerrorrepo—>Access control->select 'git-credential-osxkeychain’—>del—>save—>enter sys pwd.
```sh
$git pull   /* (Deny the given notification) will get already up to date
```
### CodeCommit Basics
#### 1.Create, View, Edit, and Delete a Repository
- AWS Console—>CodeCommit—>create new repo—>name “LatestRepo” (must be unique), description  “test repo” —>create.
- If we click on created repo, we can view code,branches,settings,etc…
- If we want to delete repo, Settings->delete repo.
- To do the same on  AWS CLI:
```sh
( “CLI-Repo” is a repo name here we are using )
$aws codecommit create-repository --repository-name CLI-Repo --repository-description “My description”  /* to create a “CLI-Repo” repo with description
$aws codecommit list-repositories    /* to list repos
$aws codecommit get-repository --repository-name CLI-Repo /* to get one repo
$aws codecommit batch-get-repositories --repository-name <Repo1> <Repo2> /* to get one json formatted info of two or more repos
$aws codecommit update-repository-name --old-name CLIRepo --new-name newReponame /* to change repo name, can check it out by ‘list-repos’ cmnd
$aws codecommit update-repository-description --repository-name CLIRepo --repository-description “update description" /* to change repo description can check it out by ‘get-repo’ command
$aws codecommit delete-repository --repository-name CLI-Repo  /* to del repo
```
#### 2.Cloning Repositories, Commits, Push, and Pulls
```sh
$aws codecommit  list-repositories
```
- Assume that there are 3 users, User1 & User2 & User3
- To clone a repo:
  - AWS—>CodeCommit—>select repo name “Myrepo"—>code—>clone URL—>select SSH/HTTP url
- In CLI:
```sh
- Here “Myrepo” is the central repo
- Here User1 making changes to “Myrepo” by cloning into his local repo
$git clone <url> <Myreponame> <My-local-repo-name>
$ls  /*to check local repo cloned/not
$cd <My-local-repo-name>
$ls   /* lists files which were created in “Myrepo”, which cloned now
- Now make some changes in local repo by creating a file “what_is_www”
$touch what_is_www
$nano what_is_www   /* to edit file
#!/bin/bash
#My first script
echo “hello"
save & exit
$chmod 755 what_is_www  /* changing permissions to execute
$./what_is_www
hello
$git status  /*tells us that there are untracked files “what_is_www”
$git add what_is_www   /* to add file to commit
$git status     /* gives changes committed
$git rm --cached what_is_www /* to remove file, if it added by mistake
$git status  /* again gives, untracked files
$git add what_is_www  /* so again add it if we want that file & check status
$git commit -m “adding file to central repo” /* asks for email & name if not configured
-To configure user globally
$git config --global user.email “username@Myreponame”
$git config --global user.name “username”
$git commit -m “adding file to central repo”  /* Now commit changes
$git remote    /* to get remote name
$git branch    /* to get branch name
$git diff --stat <remote name> <branch name>   /*to view which files to be pushed
$git push <remote name> <branch name>  
```
- After push success, check file pushed to “MyRepo” or not
- Goto AWS—>CodeCommit—>select repo—>“MyRepo”—>“what_is_www” file will be there.

- Now User2 has “Myrepo” with .txt files but there is no “what_is_www” file in it.
- So User2 has to clone the “Myrepo” with new changes i.e.,with “what_is_www” file
```sh
$ls    /* lists “User2-local-repo” dir ,which already cloned from “Myrepo” to local repo before changes pushed by User1
$cd User2-local-repo
$ls    /* files will be listed
- Now User2 has to Pull the new changes from “Myrepo” which are pushed by User1
$git remote
$git branch
$git pull <remote name>  <branch name>
$ls   /* lists what_is_www file along with previous files
$./what_is_www    /* gives info which is in file
Now User1 & User2 both have central repo with same files.
```
#### 3.Merging Basic Conflicts
-Consider if User1 & User2 added some changes to “What_is_www” file
```sh
User1:
$cd <My-local_repo>
$nano What_is_www
#!/bin/bash
#My first script
echo “hello”  User1 Conflict
save & exit
$git add what_is_www
$git commit -m “User1 conflicts added to file"
$git push <remote name>  <branch name>
-To verify Goto  AWS—code commit—Myrepo—what_is_www—(we can see the edited  file with User1 conflict)

User2:
$cd <User2-local_repo>
$nano What_is_www
#!/bin/bash
#My first script
echo “hello”  User2 Conflict
save & exit
$git add what_is_www
$git commit -m “User2 conflicts added to file"
$git push <remote name>  <branch name>  /*gives error and asks to merge changes
$git pull <remote name>  <branch name>  /*gives conflict automatic merge failed error
$nano What_is_www   /*can view file with HEAD pointer & commit-id
#!/bin/bash
#My first script
>>>>HEAD
echo “hello”  User2 Conflict
- - - - - -
echo “hello”  User1 Conflict
>>>>> <commit-id>
-Now decide which one wants to be in the file and del remaining part.Here we are deleting User1 conflict
#!/bin/bash
#My first script
echo “hello”  User2 Conflict
save & exit
-Again git add,commit & push the changes
$git add what_is_www
$git commit -m “Overwriting User1 conflict"
$git push <remote name>  <branch name>  
```
- To verify Goto  
  - AWS—>CodeCommit—>Myrepo—>what_is_www—(we can see the edited file with User2 conflict)
- Now User1, doesn’t have the latest changes in the file. So just pull the changes to User1 local repo
```sh
$git pull <remote name>  <branch name>
$nano what_is_www  /*can view the User2 conflict which changed by User2.
-Now User1 & User2 both have updated repos
```
#### 4.Branches (local)
- Assume we have User1,User2 & User3
```sh
$cd User3_local repo   
$ls    /*which has the updated repo with files same as User1, User2
- Now User3 wants to make changes by developing a new feature. so User3 creates a new branch “new_feature”
$git checkout -b new_feature   /*creates new branch & switches into it
$touch  widgets.html  
$nano widgets.html
$git branch  /*lists all branches,* points to the current branch
$git branch -r /*lists the remote branches
$git checkout <new branch name>  /*switch to another branch
$nano widgets.html  /*edit file by adding
Testing Branches /*add this line in <body> section of the file
$git checkout master  /*switching into master branch
$nano widgets.html /* we can see the changes which we made in new_feature branch.
Note: Without committing the changes to the specific branch, there will be no diff in files even we switch branches.
$git checkout new_feature  /*switching into new_feature
$nano widgets.html  /*Replace "Testing Branches" line with “new_feature branch only”
$git add widgets.html
$git commit -m “new_feature addition”  /*now changes in the file committed
$git checkout master /* Now if we change branch & check the file
$nano widgets.html /*file will be empty, since file is no longer  with this branch
$ls    /*there will be no “widgets.html” file in this branch
```
- Now to incorporate  the changes to “master” branch, we need to pull the “new_feature” branch changes & merge them into “master” branch
- To do this pull, we need to be in “master” branch
Note: we must be in the branch, that we want to merge the other branch into.
- Already we are in master branch,so do merge
```sh
$git merge new_feature
$ls  /*now we can see the “widgets.html” .
- Now we have merged the changes in to master branch, so we no longer need the new_feature branch
$git branch -d new_feature  /* to delete “new_feature” branch
$git branch   /* can check whether branch deleted/not
$git push origin master    /* Now we have to push the changes from local repo to central repo [origin=remote name,master=branch name ]
```
- we can check this by going to
AWS->CodeCommit->“My repo”->widgets.html with “new_feature branch only” line
#### 5. A Closer look at Commits and the HEAD Pointer
```sh
$cd User3_local repo   
$git branch   /*lists master branch
$ls     /*lists widgets.html file
```
- If we switch to another branch, then that branch also points to the same commit-Id of
the previous branch. so lists same files as previous one.i.e,.
```sh
$git checkout -b new_feature   /*created & switched branch
$ls  /*lists widgets.html file only
$git branch  /*lists branches with a * points to current branch i.e, HEAD points to that branch whichever branch has *. If we switches branches then HEAD pointer also switches.
```
- If we add a new file “newfeature.txt" in “new_feature” branch & do git add & commit. AND add a new file “master.txt" in “master” branch & do git add & commit.
- If we check, new_feature branch—>ls—>newfeature.txt
  AND master branch—>ls—>master.txt
- Now merge the changes being in master branch, then if we do ls, we can see  newfeature.txt,master.txt files. so we can delete new_feature branch now.
#### 6.Tags
- To manage Tags locally
```sh
$cd User3_local repo   
$git log   /* gives detailed info of commits with commit-Id’s & details
$git tag v1.0 <1st 7 letters of commit-id> /* to tag a commit with tag name “v1.0”
$git tag       /* to view our tag
$git show  v1.0   /* to see detailed info of created tag
$git tag -d  v1.0    /* to delete the tag
$git tag    /* doesn’t list anything, since we deleted tag v1.0
```
- To manage Tags with Central Repo
```sh
$git log     
$git tag v5.0 <commit-id>
$git push origin v5.0   /*Push to add tag to central repo here origin=remote name, v5.0=tag name
$git ls-remote - -tags    /*gives tags in central repo
$git fetch  - -tags   /*to pull tags from central repo to local repo
$git push - -delete origin v5.0 /*deletes tag from central repo
$git ls-remote - -tags    /*there is no v5.0 tag in central repo
$git tag     /*it will show v5.0,since it is in local repo which we pulled to local repo using fetch command
```
#### 7.Migrating a Repository into CodeCommit
- AWS allows us to migrate other git-based repos seamlessly into CodeCommit
    - Github
    - Beanstalk
    - GitLab
    - BitBucket,...
- We can choose to migrate the entire repo or just some of the branches
- We can migrate from other(non-git-based) version control systems such as Perforce,Subversion or TFS, but we have to migrate to a git-based system first.
- Migrating:
  1. Create a new repo in AWS CodeCommit
  2. Clone the repo we want to migrate to our local machine(into a temp dir) from GitHub
  3. Push the cloned GitHub repo on our local machine to the repo we created in AWS CodeCommit during step1
  4. Delete the temp dir that housed the cloned GitHub repo on our local machine
  5. Clone the AWS CodeCommit repo that we migrated to our local machine
```sh
$aws code commit create-repository --repository-name MyMigratedrepo --repository-description “my migrated repo”
```
- Goto AWS—CodeCommit—MyMigratedRepo
```sh
$git clone --mirror <repo to migrate url>  <my local temp dir>
```
- Goto GitHub—login to our account—select the Repo which we want to migrate—Copy the URL to clone.
```sh
$git clone - -mirror <paste URL>  mytempdirformigration
$ls   /* our tmp dir has to be listed
$cd mytempdirformigration   /*cd to push our cloned repo to AWS
$git push https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyMigratedRepo  --all     /*can copy URL from AWS CodeCommit
```
- Goto AWS—CodeCommit—MyMigratedRepo—[have to see the files which cloned from selected GitHub Repo]
```sh
$cd ..
$rm -rf  mytempdirformigration   /*delete the temp dir
$ls    /*check dir deleted/not
$git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyMigratedRepo  <local repo name>
$git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyMigratedRepo  my-local-migrated-repo
$ls    /* local dir has to be listed
$cd my-local-migrated-repo
$ls   /*has to list the files which are in our AWS CodeCommit MyMigratedrepo & GitHub repo
```
### Using CodeCommit with other AWS Services
#### 1.Triggers:
- Triggers are a method where events in CodeCommit can launch automated actions in other AWS services
i.e., creating a new branch can trigger an SNS topic to notify all subscribers that a new branch has been created
- Currently, we can select one of the following events in CodeCommit to invoke a trigger:
    - All repository events
    - A push to existing branch(*master branch)
    - Creation/deletion of a branch or tag
- Triggers can invoke actions in the following AWS services:
    - AWS SNS
    - AWS Lambda
    - We can also take advantage of ANS’s ability to integrate with other AWS services like Simple Queue Service(SQS)

#### 2.CodeCommit Trigger for SNS
##### SNS Triggers: Setup/Getting Started:
- Triggers can be setup using both
    - AWS Console
    - AWS CLI
- Prerequisites for SNS Triggers:
    - Must have an existing SNS topic
    - The Repository & SNS topic must be in the same region
- Cross-Account Triggers
    - If setting up repos & SNS topics using different root accounts(not different users within the same account),we must have the proper permissions set so CodeCommit can communicate with SNS in the other account.

- Now create a new repo,to test Trigger on
  - AWS Console—->CodeCommit—->Create new repo—->“TriggerRepo"—->Create

- We need to create a SNS Topic, to setup a Trigger with
  - AWS Console—->SNS—Create Topic—->"Trigger Topic"—->Create Topic—->Create Subscription—->Protocol—->Email—->Email-->Id—->Create subscription—->confirm subscription by going to Email-Id.

- Goto CodeCommit Repo,Create a Trigger & Test it
  - AWS Console—>CodeCommit—>Create Trigger—>Trigger name—>“NewBranchTrigger”—>Events—>Select Create a branch or tag—>Branch name—>All Branches—>Send to—>Amazon SNS—>SNS Topic—>“Trigger Topic”—>Test trigger (to test, check email)—>Create

- Clone TriggerRepo to local machine
- Goto CLI,consider this doing on User1 m/c
```sh
$git clone <copy URL from CodeCommit TriggerRepo>  Local-Trigger-Repo
$cd Local-Trigger-Repo
$touch file.txt
$git add file.txt
$git commit -m “setting up master branch”
$git push origin master
- create a new branch & make some changes in a file
$git checkout -b TriggerBranch
$touch new.txt
$git add new.txt
$git commit -m “test commit for our trigger”
$git push origin TriggerBranch
- Now we have two notifications about branches “master” & “TriggerBranch”  created  in our Email
```

##### SNS Triggers using AWS CLI :
- Trigger JSON file “Events” list
    - all  (all events in repo)
    - updateReference  (push occurred)
    - createReference   (branch or tag created)
    - deleteReference   (branch or tag deleted)
- For creating,pushing & viewing triggers - we will use AWS CLI commands NOT Git commands.

Test Trigger before pushing it:
```sh
aws codecommit test-repository-triggers - -cli-input-json file://<JSON file>
```
Push Trigger to the Repo:
```sh
aws codecommit put-repository-triggers - -cli-input-json file://<JSON file>
```
View Triggers in the Repo:
```sh
aws codecommit get-repository-triggers - -repository-name <Repo Name>
```

Goto AWS—>SNS—>Topics—>TriggerTopic—>copy ARN
```sh
$touch RepoTrigger.json  /*creating a json file
$nano RepoTrigger.json
{
    "repositoryName": “TriggerRepo",
    "triggers": [
        {
            "name": “deleteTrigger",
            "destinationArn": “<Paste copied ARN of TriggerTopic from SNS Topic>",
            "customData": "",
            "branches": [

            ],
            "events": [
                “deleteReference"
            ]
        }

    ]
}
Save & exit
```
- Test Trigger
```sh
$aws codecommit test-repository-triggers - -cli-input-json file://RepoTrigger.json  /*to test Trigger,we will get an test email as branches deleted
```
- Push Trigger to the Repo:
```sh
$aws codecommit put-repository-triggers - -cli-input-json file://RepoTrigger.json
```
- If we goto AWS—>CodeCommit—>Triggers—>TriggerRepo—>“deleteTrigger" will be replaces in place of “NewBranchTrigger”.
- View Triggers in the Repo
```sh
$aws codecommit get-repository-triggers - -repository-name TriggerRepo /*gives JSON format of our file
```
- Now our Trigger has been successfully created,now will check by deleting one of our branches, we have to receive a notification.
```sh
$git branch
*TriggerBranch
master
$git checkout master  /*to delete TriggerBranch,we have to switch the branch
$git branch -D TriggerBranch  /*to delete it from local repo
$git push origin - -delete TriggerBranch  /*to delete branch from central (repo,origin=remote name, TriggerBranch=branch name)
```
- Now we should get the notification about TriggerBranch delete.
#### 3.CodeCommit Cross-Account SNS Trigger
- Consider there are Root Account#1, Root Account#2.
- Root Account#1 contains CodeCommit Repo “TriggerRepo” & also contains user accounts for our developers.
- Root Account#2 contains SNS Topic that we want to Trigger when Events happened in our “TriggerRepo” repository.
- To configure the Cross-Account SNS Trigger,we must first edit the SNS Policy in Root Account#2,then point our Root Account#1 to it. (Can copy the SNS Policy template from AWS-SNS-Topic-Other topic Actions-Edit Topic Policy-Advanced view& edit it as follows)
- Login into AWS Root Account#1—>CodeCommit—>“TriggerRepo”—>Triggers—>select “deleteTrigger”—>delete.Goto settings,copy Repository ARN of “TriggerRepo”.
- Now edit the SNS Policy template,
In CodeCommit section, AWS:SourceARN<paste copied ARN>, AWS:SourceAccount<number from copied ARN>     
- Now login to Root Account#2—>SNS—>Topics—>Create“CrossAccountTriggerTopic”—>Add Subscription to Email.
- Again goto to SNS Policy template,edit
Resource<Copy & paste SNS Topic ARN> in two places,  AWS:SourceOwner<copy & paste SNS Source number from ARN>
- Now copy the whole SNS Policy Template,
- Goto AWS—>SNS—>Topics—>select “CrossAccountTriggerTopic”—>other topic Actions—>Edit Topic Policy—>Advanced View—>Delete default one & Paste the SNS Policy Template—>Update policy.
- Again login to Root Account#1—>CodeCommit—>“TriggerRepo”—>Triggers—>Create Trigger—>“CrossTest”—>Events—>Create a branch/tag—>Branch names—>All branches—>send to—>Amazon SNS—>SNS Topic—>Type an Amazon SNS topic ARN—>SNS Topic ARN—<paste the SNS topic ARN of Root Account#2 which pasted in SNS Policy template>—>Test trigger  
- We should get an email notification.
#### 4.CodeCommit Trigger for Lambda Functions
AWS Lambda: is a compute service where we can upload our code to lambda & service  can run the code on our behalf using AWS infrastructure.
Prerequisites for lambda triggers:
- Must have an existing Lambda function created
- The repository & the Lambda function must be in the same region.

Creating a Lambda Trigger Using AWS
Setting up Lambda Permissions:
```sh
$aws lambda add-permission - -cli-input-json file://<My Permissions JSON File>
```
-In Permissions JSON File, we must have 'Lambda Function name’,'Repository ARN’,'Repository Source Number’.

AWS Console—Lambda—Create Function—Type ‘hello-World’ function in Filter option —> click on hello world—> ‘hello World’ [code will be provided by ‘hello World’ function
Handler as ‘index.handler’ which is in code] Name “CodeCommitTriggerFunction" —> Role as ‘create new role from templates—>Role name as ‘lambda_basic_execution’ —> Policy templates as ‘simple micro service permissions’ —>
Memory as ‘128MB’ —> Timeout ‘0 min 3 sec’ —> VPC as ‘no VPC’ —> Next
Review Page  [where we can review all of the functions we have given] —> create function
If we want to test the code, click on TEST & see results.
If we want to change the values used in Test,  goto Actions —> configure test events —> save & test —> check Log output  [where we can see console log msgs]
Execution result is the output of the returned ‘callback’  function.

AWS—CodeCommit—Create a new repo—“LambdaTriggerRepo”—create—Settings—Repository ARN(copy that for Permissions JSON file).

Goto CLI, User1
$touch CodeCommitLambdaPermission.json
$nano  CodeCommitLambdaPermission.json
{
“FunctionName”: “CodeCommitTriggerFunction”,
“StatementId”: “1”,
“Action”: “lambda:InvokeFunction”,
“Principal”: “codecommit.amazonaws.com”,
“SourceARN”: "< Paste copied Repo ARN >”,
“SourceAccount”: “<Repo Source number from copied ARN >"
}
Save & exit

$aws lambda add-permission - -cli-input-json file://CodeCommitLambdaPermission.json    /*which gives our file content as output

-Add Permissions to the role which is created while creating a lambda function
-AWS—IAM—Roles—select “lambda_basic_execution”—Permissions—Add Inline Policy--create role policy—Policy generator—select—Edit Permissions—Effect “Allow”—AWS Service—AWSCodeCommit—Actions—Get Repository—Amazon Repository Name(ARN)—Paste copied Repo ARN—Add statement—next step—Apply Policy.

-Now create a CodeCommit Trigger
-AWS—CodeCommit—LambdaTriggerRepo—Triggers—Create Trigger— “LambdaTrigger”—Events—All Repository Events—Banch name—All Branches—send to — AWS Lambda—Lambda Function—“CodeCommitTriggerFunction”—Test Trigger—If Test successful—create.

-Now to verify the test is actually works
AWS—Lambda—“CodeCommitTriggerFunction”—Monitoring—View logs in cloud watch—click on latest log stream(which should see clone URL of our LambdaTriggerRepo & References: ref/head/Test Reference)

-Now goto CLI,
$cd local-lambda repo
$git clone <Copy & Paste the URL of LambdaTriggerRepo from CodeCommit  into “local-lambda-repo” dir>
$touch file.txt
$git add file.txt
$git commit -m “testing trigger”
$git push origin master  (ie., <remote name> <branch name>)

Again check logs in cloud watch
AWS—Lambda—“CodeCommitTriggerFunction”—Monitoring—View logs in cloud watch—click on latest log stream(which should see References: ref/head/Test Reference & References: ref/head/master & Clone URL)

Create a Lambda Trigger using CLI
$cd local-lambda repo
$touch RepoTrigger.json
$nano RepoTrigger.json
{
    "repositoryName": “LambdaTriggerRepo",
    "triggers": [
        {
            "name": “CLITestTrigger",
            "destinationArn": “<Paste copied ARN of CodeCommitTriggerFunction>",
            "customData": "",
            "branches": [

            ],
            "events": [
                “all"
            ]
        }

    ]
}
save & exit
-Test Trigger
$aws codecommit test-repository-triggers - -cli-input-json file://RepoTrigger.json
-Push Trigger to central repo
$aws codecommit put-repository-triggers - -cli-input-json file://RepoTrigger.json
-View Triggers in the Repo
$aws codecommit get-repository-triggers - -repository-name LambdaTriggerRepo
/*gives CLITestTrigger info in output

Now create a file, add & commit that file,push that commit to activate Trigger in real world situation
$touch new.txt
$git add new.txt
$git commit -m “testing trigger”
$git push origin master

AWS—Lambda—CodeCommitTriggerFunction—Monitoring--View logs in cloud watch—click on latest log stream(which should see two separate References: ref/head/master  & References: ref/head/master & Clone URLs)
