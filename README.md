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

##### Creating a Lambda Trigger Using AWS
Setting up Lambda Permissions:
```sh
$aws lambda add-permission - -cli-input-json file://<My Permissions JSON File>
```
- In Permissions JSON File, we must have 'Lambda Function name’,'Repository ARN’,'Repository Source Number’.

AWS Console—>Lambda—>Create Function—>Type ‘hello-World’ function in Filter option —> click on hello world—> ‘hello World’ [code will be provided by ‘hello World’ function
Handler as ‘index.handler’ which is in code] Name “CodeCommitTriggerFunction" —> Role as ‘create new role from templates—>Role name as ‘lambda_basic_execution’ —> Policy templates as ‘simple micro service permissions’ —>
Memory as ‘128MB’ —> Timeout ‘0 min 3 sec’ —> VPC as ‘no VPC’ —> Next
Review Page  [where we can review all of the functions we have given] —> create function
If we want to test the code, click on TEST & see results.
If we want to change the values used in Test,  goto Actions —> configure test events —> save & test —> check Log output  [where we can see console log msgs]
Execution result is the output of the returned ‘callback’  function.

AWS—>CodeCommit—>Create a new repo—>“LambdaTriggerRepo”—>create—>Settings—>Repository ARN(copy that for Permissions JSON file).

Goto CLI, User1
```sh
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
```

- Add Permissions to the role which is created while creating a lambda function
- AWS—>IAM—>Roles—>select “lambda_basic_execution”—>Permissions—>Add Inline Policy-->create role policy—Policy generator—>select—>Edit Permissions—>Effect “Allow”—>AWS Service—>AWSCodeCommit—>Actions—>Get Repository—>Amazon Repository Name(ARN)—>Paste copied Repo ARN—>Add statement—>next step—>Apply Policy.

- Now create a CodeCommit Trigger
- AWS—>CodeCommit—>LambdaTriggerRepo—>Triggers—>Create Trigger—> “LambdaTrigger”—>Events—>All Repository Events—>Banch name—>All Branches—>send to —> AWS Lambda—>Lambda Function—>“CodeCommitTriggerFunction”—>Test Trigger—>If Test successful—>create.

- Now to verify the test is actually works
AWS—>Lambda—>“CodeCommitTriggerFunction”—>Monitoring—>View logs in cloud watch—>click on latest log stream(which should see clone URL of our LambdaTriggerRepo & References: ref/head/Test Reference)

- Now goto CLI,
$cd local-lambda repo
$git clone <Copy & Paste the URL of LambdaTriggerRepo from CodeCommit  into “local-lambda-repo” dir>
$touch file.txt
$git add file.txt
$git commit -m “testing trigger”
$git push origin master  (ie., <remote name> <branch name>)

Again check logs in cloud watch
AWS—>Lambda—>“CodeCommitTriggerFunction”—>Monitoring—>View logs in cloud watch—>click on latest log stream(which should see References: ref/head/Test Reference & References: ref/head/master & Clone URL)

Create a Lambda Trigger using CLI
```sh
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
```
- Test Trigger
```sh
$aws codecommit test-repository-triggers - -cli-input-json file://RepoTrigger.json
```
- Push Trigger to central repo
```sh
$aws codecommit put-repository-triggers - -cli-input-json file://RepoTrigger.json
```
- View Triggers in the Repo
```sh
$aws codecommit get-repository-triggers - -repository-name LambdaTriggerRepo
/*gives CLITestTrigger info in output
```

Now create a file, add & commit that file,push that commit to activate Trigger in real world situation
```sh
$touch new.txt
$git add new.txt
$git commit -m “testing trigger”
$git push origin master
```
AWS—>Lambda—>CodeCommitTriggerFunction—>Monitoring-->View logs in cloud watch—>click on latest log stream(which should see two separate References:ref/head/master  & References:ref/head/master & Clone URLs)
#### 5.CodeCommit & AWS KMS (encryption)
AWS Key Management System (KMS)
- Amazon’s encryption management service that allows us to easily create and control the keys used to encrypt our data.
- We can:
    - Create, describe, enable and disable master keys
    - Set master key usage polices and logs
    - Encrypt and decrypt our data using the master keys we create
CodeCommit & KMS
- Key creation:
    - AWS automatically creates an AWS-managed key when we create a CodeCommit repository in a given region.
    - The key is stored in the same region as the repository.
    - The key is stored in our AWS account.
- Key usage:
    - The key (created specifically for CodeCommit), is then used to encrypt and decrypt our data while both In-Transit, and when at rest.
    - In-Transit:
        - AWS encrypts the data when we push to a repository (git push)
        - AWS decrypts the data when we pull from a repository (git pull)
- Encryption Type:
    - AWS KMS uses AES-256
NOTE: CodeCommit integrates with AWS KMS automatically. we do not have to take any steps by ourself, to encrypt our data when using CodeCommit (while in-transit or at rest on AWS servers)
CodeCommit, KMS & IAM Policies:
 - CodeCommit performs these actions against the default key (aws/codecommit)
    "kms:Encrypt”
    "kms:Decrypt”
    "kms:ReEncrypt”
    "kms:GenerateDataKey”
    "kms:GenerateDataKeyWithoutPlaintext”
    "kms:DescribeKey”

AWS—>IAM—>Encryption Keys—>aws/codecommit(which will be created automatically when we create CodeCommit Repo).
### AWS CodeDeploy
#### 1.CodeDeploy
- CodeDeploy is an AWS service for automating the deployment process of our applications, from Git-based version control systems or S3 buckets to Amazon EC2 instances, on-premise instances, or both.
- Automation:
    - We can easily automate our code deployment to development, test, and production environments.
- Scale:
     - Deploy our code to one or thousands of instances at once.
- Reduced Downtime:
     - Rolling updates allow us to decrease downtime by allowing us to track application heath and stop/rollback deployments if there are errors.
- Control:
    - Easily keep track of our deployments by receiving reports that list when and where each of our application revisions is deployed.
- Platform-agnostic:
    - CodeDeploy is built to work with any application.
#### 2.Setup and Configuration (Roles and Policies)
1. Provision an IAM user with a custom CodeDeploy Policy
- Gives a non-admin user the rights to manage all the elements needed to use CodeDeploy.
  - AWS—>IAM—>Policies—>Create Policy—>Create Own Policy—>“CodeDeployCustomUser”—>Policy Document (Paste the following code in policy document)—>create Policy
```sh
{
  "Version": "2012-10-17",
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : [
        "autoscaling:*",
        "codedeploy:*",
        "ec2:*",
        "elasticloadbalancing:*",
        "iam:AddRoleToInstanceProfile",
        "iam:CreateInstanceProfile",
        "iam:CreateRole",
        "iam:DeleteInstanceProfile",
        "iam:DeleteRole",
        "iam:DeleteRolePolicy",
        "iam:GetInstanceProfile",
        "iam:GetRole",
        "iam:GetRolePolicy",
        "iam:ListInstanceProfilesForRole",
        "iam:ListRolePolicies",
        "iam:ListRoles",
        "iam:PassRole",
        "iam:PutRolePolicy",
        "iam:RemoveRoleFromInstanceProfile",
        "s3:*"
      ],
      "Resource" : "*"
    }    
  ]
}
```

IAM—>Users—>Create user “Matt"—>Permissions—>Attach Policy—> “AWSCodeCommitFullAccess”--> “CodeDeployCustomUser”—>Attach Policy.

2. Create an Instance Profile
- This allows you to launch EC2 instances that are configured for use with CodeDeploy.
  - Create another Instance Role Policy which gives access to get & list in all S3 Buckets
- AWS—>IAM—>Policies—>Create Policy—>Create Own Policy—>“CodeDeployDemo->EC2->Permissions”-->Policy Document (Paste the following code in policy document)—>create Policy
```sh
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:Get*",
        "s3:List*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```
- This Policy is attached to EC2 instance when it is running using Role
  - IAM—>Roles—>Create New Role—>"CodeDeploDemo-EC2"—>Amazon EC2—> “CodeDeployDemo->EC2->Permissions”—>Create Role.

3. Create a Service Role
- This will allow CodeDeploy to communicate and interact with other AWS Services
  - IAM—>Roles—>Create New Role—>“CodeDeployServiceRole”—>Amazon CodeDeploy—>Attach Policy—>AWSCodeDeployRole—>Create Role
4. Install the AWS Command Line Interface (CLI)
#### 3.Code Deploy Basics
##### a.Configuring an EC2 Instance for Use with CodeDeploy
- Type of instances CodeDeploy support:
Amazon Linux, Ubuntu Server, Red Hat Enterprise Linux (RHEL), Windows Server
- Configuration Steps:
1) Launch a new Amazon Linux AMI
2) Select appropriate instance Type
3) Set IAM role to the Instance Profile we created in the Setup & Configuration lesson
4) Open ‘Advanced Details’ and add the following bash script:
```sh
#!/bin/bash
yum -y update
yum install -y ruby
yum install -y aws-cli
cd /home/ec2-user
aws s3 cp s3://<bucket-name>/latest/install . --region <region-name>
chmod +x ./install
./install auto
```
5) Add storage
6) Add a ‘Name’ Tag to the instance
7) Configure a Security Group
8) Review and Launch
9) Select a key pair
10) Check to see if the AWS CodeDeploy agent has been successfully installed
##### AWS CodeDeploy Agent:
The AWS CodeDeploy agent is a custom software package that must be installed on all instances that will be part of a deployment group.
 - The agent specifies many of the settings that are needed for the instance to interact with CodeDepoy – like directory paths, log files, and deployment polling time intervals.
- The agent is also customizable, so we can alter it to fit our deployment needs.
For Amazon, Ubuntu, and RHEL instances:
  - Name: codedeployagent.yml
  - Location: /etc/codedeploy-agent/conf
  - S3 Bucket Name: aws-codedeploy-us-east-1    if region: us-east-1, (replace it with in whichever region we want to launch our EC2.)  

##### CodeDeploy Agent Status:
Command:
```sh
sudo service codedeploy-agent status
```
- Good Response: The AWS CodeDeploy agent is running.
- Error Responses:
    - Error: No AWS CodeDeploy agent running
```sh
    - Fix: sudo service codedeploy-agent start
```
- Error: codedeploy-agent: unrecognized service
  - Fix: Launch a new instance and double check the bash script for errors, OR double check the permissions policy attached to the Instance Profile (make sure it allows access to S3)

- AWS—>EC2—>SG—>create a SG—>“CodeDeploySG” with SSH,HTTP Allow Rules
- AWS—>EC2—>Amazon Linux AMI—>Configure Instance—>IAM Role—>"CodeDeploDemo-EC2”—>Advanced Details—>Copy & paste above bash script [Edit Bucket name: aws-codedeploy-us-west-1  & Region Name: us-west-1]—>Next—>Next—>Tag—>Name->“CodeDeployTestInstance”—>Configure SG—>select existing SG->“CodeDeploySG”—>review & launch—>create new key->pair & download it.
- After EC2 running,SSH into it to connect through CLI
```sh
$sudo service codedeploy-agent status /*to see the agent status
```
##### b.Creating an Application, Deployment Group and Deployment Configuration
- CodeDeploy Application:
  - The application is simply a name 'identifier' used to reference our deployment settings
- CodeDeploy Deployment Group:  
  - The deployment group is the 'instance' (or instances) we want to 'target and deploy' our code to
- CodeDeploy Deployment Configuration:
  - Select 'how many instances' in our deployment group our code will deploy to at 'any given time'

##### Managing an Application from the AWS Console:
- Creating an Application, Deployment Group & Configuration:
1) Navigate to CodeDeploy
2) Choose Create a New Application
3) Give the Application a name
- The next set of steps is to set up the Deployment Group within the Application:
4) Give the deployment group a name
5) Select the tag type, Key & value for your instance
- The next step is to setup the Deployment Configuration within the Application:
6) Select deployment configuration:
    - OneAtATime, AllAtOnce, HalfAtATime
- Optional:
7) Create a trigger
- Set permissions:
8) Select the CodeDeploy Service role we create
9) Finalize & create the Application

AWS—>CodeDeploy—>Custom Deployment—>Skip Walkthrough—>Application name “Test Application”—>Deployment group name “TestDeploymentGroupName”—>Deployment type—>select accordingly—>Environment config—>Amazon EC2 Instances—>key “name”—>value “CodeDeployTestInstance”—>Deployment config—>OneAtATime—>Service Role—>select ARN of “CodeDeployServiceRole”—>create application

##### Managing an Application from the AWS CLI:
- Creating an Application, Deployment Group & Configuration:
- Base CLI Command:
    ``` aws deploy ```
1) Create a new application
    ```aws deploy create-application --application-name```
2) Create the deployment group, configuration & other options
- In one command we will specify:
    - Deployment group (tag, key, value)  
    - Deployment Configuration (AllAtOnce, OneAtATime, HalfAtATime)
    - Trigger (optional)
    - Service role (permissions)
```sh
 aws deploy create-deployment-group --application-name <NAME> --deployment-group-name <NAME> --ec2-tag-filters Key=< >,Value=< >,Type=KEY_AND_VALUE --deployment-config-name CodeDeployDefault.<SELECTOPTION> --service-role-arn <SERVICE-ROLE_ARN>
```

```sh
$aws deploy create-application --application-name CLIApplication /*creates application
```
To verify, goto AWS—>CodeDeploy—>Applications—>CLIApplication will be created

```sh
$aws deploy create-deployment-group --application-name CLIApplication - -deployment-group-name CLIDeploymentGroup --ec2-tag-filters Key=Name,  Value=CodeDeployTestInstance, Type=KEY_AND_VALUE --deployment-config-name CodeDeployDefault.AllAtOnce  --service-role-arn <Copy “CodeDeployService” ARN & Paste it>  /*creates Deployment Group
```

[In Above Command:
-application-name <NAME>: “CLIApplication" which we created in 1st command
-EC2-Runing instance-Tags-Copy Key & Value details & paste it
ie..key = name  & value = “CodeDeployTestInstance”
-deployment-group-name <NAME>: “CLIDeploymentGroup” /*can give any name
-CodeDeployDefault.<SELECTOPTION>:AllAtOnce /*mentioning one option  
-<SERVICE-ROLE_ARN> : IAM-Roles-“CodeDeployService”-Copy ARN& Paste it]

- To verify, goto AWS—>CodeDeploy—>Applications—>CLIApplication—>Deployment Group->
 CLIDeploymentGroup will be created with given options.
##### c.Editing and Deleting an CodeDeploy Application
- Changing the Application Name:
    - AWS Console: N/A
    - AWS CLI Commands:
    ```
    aws deploy list-applications        (list all our applications)
    aws deploy update-application --application-name <NAME> --new-application-name <NEW NAME>       (rename our application)
    ```
 - Deleting an Application:
    - AWS CLI Commands:
    ```
    aws deploy delete-application --application-name <NAME> (delete an application)   
    ```
      - AWS Console:
        1) Click on the application
        2) Scroll to the bottom and click ‘Delete Application’
```sh
 $aws deploy list-applications  /*lists applications
 $aws deploy update-application --application-name CLIApplication --new-application-name CLIChangeApplication    /*Renames CLIApplication to CLIChangeApplication
 $aws deploy list-applications    /*can see the changed name here
 $aws deploy delete-application --application-name CLIChangeApplication /*to delete
```

 To delete an application from AWS Console,
 - AWS—>CodeDeploy—>Applications—>select application—>Delete application

 ##### d.Adding, Editing, and Deleting Application Elements (Deployment Groups & Configurations)
 - Edit, Add & Delete Deployment Groups & Configurations (in the AWS Console):
 - Editing and Adding Groups & Configurations:
 1) Navigate to CodeDeploy
 2) Click on one of your applications
 3) Select a deployment group and under actions, select “Edit”
 4) Here you can:
    - Change the deployment group name
    - Add/change/delete instances to deploy to
    - Change the deployment configurations
    - Add/edit/remove SNS Triggers
    - Change the service role
 - Adding an Additional Deployment Group:
  1) Click on an application
  2) Under Deployment Groups, click ‘Create Deployment Group’
  3) Fill in the form with new information
- Deleting an Deployment Group:
 1) Click on an application
 2) Under Deployment Groups, select a deployment group
 3) Under Actions, click ‘Delete’

 AWS->CodeDeploy->Applications->"Test Application”->Deployment Groups->select group->Actions->Edit    /*we can edit whatever options we want

 AWS->CodeDeploy->Applications->"Test Application”->Deployment Groups->create Deployment group   /*to create additional deployment groups  

 AWS->CodeDeploy->Applications->"Test Application”->Deployment Groups->select group->Actions->Delete   /*to delete deployment group   

##### Edit, Add & Delete Deployment Groups & Configurations (using the AWS CLI):
 - Base CLI Command:
     ```aws deploy```
 - Edit a deployment group/configuration elements:
    - required arguments:
     ```sh
     aws deploy update-deployment-group --application-name <NAME> --current-deployment-group-name <NAME>
     ```
 - Optional (elements to change):
    ```sh
     --new-deployment-group-name <NAME>
     --ec2-tag-filters
     --on-premises-instance-tag-filters
     Key=Name,Value=,Type=KEY_AND_VALUE
     --auto-scaling-groups <NAME>
     --deployment-config-name CodeDeployDefault.<SELECTOPTION>
     --service-role-arn <SERVICE-ROLE_ARN>
     ```
 - Delete a deployment group/configuration:
 ```sh
 aws deploy delete-deployment-group --application-name <NAME> --deployment-group-name <NAME>
```
```sh
 $aws deploy list-applications
 $aws deploy list-deployment-groups --application-name TestApplication
 $aws deploy update-deployment-group --application-name TestApplication --current-deployment-group-name TestDeploymentGroupName --new-deployment-group-name NewDeploymentGroupName --deployment-config-name CodeDeployDefault.HalfAtATime
 $aws deploy list-deployment-groups --application-name TestApplication /*lists changed deployment group name
 $aws deploy delete-deployment-group --application-name TestApplication --deployment-group-name NewDeploymentGroupName  /*deletes deployment group
 ```
 ##### e.Creating, Viewing, and Deleting a Custom Deployment Configuration
- Default AWS Deployment Configurations:OneAtATime , AllAtOnce , HalfAtATime
- Creating a Custom Deployment Configuration:
    - You can create a custom configuration via:
        - The AWS CLI
        - The AWS APIs
        - AWS CloudFormation Template
    - CLI Command:
        ``` aws deploy create-deployment-config```
    - Options:
    ```sh
    --deployment-config-name <NAME>
    --minimum-healthy-hosts type=<OPTION>,value=<#or%>
    -HOST_COUNT
    -FLEET_PERCENT
    ```
- Viewing a Custom Deployment Configuration:
    - CLI Commands:
    - To list all Deployment Configurations
    ```aws deploy list-deployment-configs```
    - To view detailed information on a specific Configuration
    ```sh
    aws deploy get-deployment-config --deployment-config-name <NAME>
    ```
- Deleting a Custom Deployment Configuration:
    - CLI Command:
    ```aws deploy delete-deployment-config --deployment-config-name <NAME>```
```sh
$aws deploy create-deployment-config --deployment-config-name MyCustomConfig --minimum-healthy-hosts type=HOST_COUNT,value=1
$aws deploy list-deployment-configs    /*lists configs
$aws deploy get-deployment-config --deployment-config-name MyCustomConfig
$aws deploy delete-deployment-config --deployment-config-name MyCustomConfig
```
##### f.Creating and Configuring the AppSpec File
- AppSpec (short for Application Specification):
- Is YAML-formatted file used to specify the:
    - Source and target location(required) of the files we want to deploy
    - Permissions(optional) given to your files once at their target location
    *Only applies to Amazon Linux, Ubuntu Server & RHEL
    - Lifecycle event hooks(optional) available to run specific scripts against.   
     Events in the deployment lifecycle that can trigger specific scripts to run
The AppSpec file MUST be named “appspec.yml”
- AppSpec File “Files” Section:
- Executes during the deployments “Install” lifecycle event.
- Source Options:
  - If ‘source’ refers to a file, ONLY that file will be installed
  - If ‘source’ refers to a directory, ALL directory content will be installed
  - If ‘source’ is just a single ‘ / ’, ALL files in the Revision will be installed

Lets Consider the following is our tree structure:
- /local-MyRepo
    - .txt file
    - html files [ index.html (777) , wonder widgets.html(777) ]
    - executable files [ what_is_www ]
- Goto CLI, User1 login
```sh
$cd local-MyRepo
$ls   /*executables .html files & .txt files will be listed
$touch appsec.yml   /*must be specified in our root dir of our src files
$nano appsec.yml    /*to edit file. [check appsec.yml file in github]
```
##### g.Creating and Uploading a Revision to S3
Application Revision:
- Revision is simply the bundle(zip/tar file) containing the current group of files that we want to deploy.
1. select a repository type : where we want to store our bundled src files, so CodeDeploy can access them for deployment.
- CodeDeploy can be setup to access src files from AWS S3 Bucket,GitHub,BitBucket
2. Bundle(.zip)our src files &push(upload) it to the repo we select (above) to use.

- we have to push our src files from CodeCommit to an S3 bucket, & deploy our files from there.
##### Pushing Revision to an S3 Bucket:
1. create an S3 Bucket for our revisions
2. Bundle & upload the src files to S3 bucket
```sh
aws deploy push --application-name <NAME> --description <“description”> --ignore-hidden-files --s3-location s3://<Bucket-Name>/<File-Name>.zip --source .
```

AWS—>S3—>create a bucket “wonder-widgets"
- Goto CLI,User1
```sh
$cd local-MyRepo
$ls   /*lists appsec.yml , html files, executable files, .txt files
$aws deploy push --application-name TestApplication --description “test deployment”  --ignore-hidden-files --s3-location s3://wonderwidgets-deployment-test/wonderwidgets.zip --source .
```
- We can check it by going to :
AWS—>S3—>wonderwidgets-deployment-test—>wonderwidgets.zip

##### h.Deploying a Revision from Amazon S3
We can deploy a revision to an instance via:  
- AWS Console.
- AWS CLI
- AWS API
##### Deploying a Revision via the AWS Console:
- AWS—>CodeDeploy—>Get started—>custom deployment—>skip walk through—>Select Deployments (from AWS CodeDeploy drop down at the top of the dashboard) —>Application—>“TestApplication”(which we created above)—> Deployment group—>“TestDeploymentGroupName”—>Revision type—>select "My application is stored in Amazon S3”—>revision location—> "s3://<wonderwidgets-deployment-test/wonderwidgets.zip”—>description—>“test”—> Deployment configuration—>select “CodeDeployDefault:AllAtOnce”—>Deploy Now—>check status
- AWS—>EC2—>CodeDeployTestInstance(running)—>Connect it through SSH
```sh
$ssh into ec2
$cd /
$ls   /*check “wonderwidgets” created/not
$cd wonderwidgets
$ls  /*lists appsec.yml,executables,html,.txt files  
$cd html
$ls -l /* permissions of html files must be 400 except index.html
$cd /
```
##### Deploying a Revision using the AWS CLI:
- Now open another CLI terminal, login as User1
```sh
$cd local-MyRepo
$nano appsec.yml /*edit mode=644 instead of 400
$aws deploy push --application-name TestApplication --description “second test” --ignore-hidden-files --s3-location s3://wonderwidgets-deployment-test/wonderwidgets.zip --source .
```
- Gives create-deployment command as o/p,run that command as follows
```sh
$aws deploy create-deployment --application-name TestApplication --s3-location bucket=wonderwidgets-deployment-test,key=wonderwidgets1.zip,bundleType=zip,eTag=<S3 Ref Num> --deployment-group-name TestDeploymentGroupName --deployment-config-name CodeDeployDefault.OneAtATime --description “test2”
```
- Gives Deployment ID as o/p,copy & paste it in the following command
- Now check the deployment status
```sh
$aws deploy get-deployment --deployment-id <ID>
$aws deploy get-deployment --deployment-id <Paste the copied deployment-id>
```
- Now Goto EC2 instance connected terminal
```sh
$cd wonderwidgets
$cd html
$ls -l  /* permissions of html files must be 644 except index.html
```
##### i.Monitoring Deployments via SNS Triggers
- SNS Triggers must be setup pre-deployment
- SNS Triggers are great for automating the monitoring process. Allowing us to get instant notifications if deployments or instances fail or succeed.
- SNS Triggers can be setup via:
    - AWS Console
    - AWS CLI
- SNS Triggers:
    - Deployment Status: Deployment Starts, Deployment succeeds , Deployment fails , Deployment Stops
    - Instance Status:  Instance Starts , Instance Succeeds , Instance Fails
##### Add SNS Triggers via AWS Console:
- Must have a SNS Topic “Trigger Topic" created & subscribed to our Email
- AWS—>CodeDeploy—>click on “Test Application”(which we created before)—>open up the details of our “TestDeploymentGroupName”—>Triggers—>Create Trigger—>Trigger Name—>“CodeDeployTrigger”—>Events—>select “Deployments status(All)—>Amazon SNS Topic—>select “Trigger Topic”(which is created before & subscribed to our Email.If not created already, create it first & then add SNS triggers)—>create Trigger
- By clicking on the edit option/icon beside Trigger, we can edit or delete Trigger.
- Now run a deployment,& check whether we get a notification from our SNS Trigger when our deployment fails
- Before creating deployment,upload new revision location into S3 bucket "s3://<wonderwidgets-deployment-test/wonderwidgetsFAIL.zip” by following the "Deploying a Revision via the AWS CLI: Deployment creation"
- AWS CodeDeploy—>Deployments(from dropdown list at the top of dashboard)—>create a new deployment—>Application—>select “Test Application”—>Deployment Group—>select “TestDeploymentGroupName”—>Revision Type—>select “My application is stored in Amazon S3”—>Revision Location—>select ""s3://<wonderwidgets-deployment-test/wonderwidgetsFAIL.zip”—>Deployment Description—>“Fail”—>Deployment Config—>select “CodeDeployDefault:OneAtATime”—>Deploy Now—>status—>failed
- If we check our Email, we get notifications about Success,Creation &Failed deployment status.
##### Creating an SNS Triggers via AWS CLI:
- Goto CLI, User1 login
a. Create a new Deployment Group:
1) Run the following command
```
$aws deploy create-deployment-group --generate-cli-skeleton /*gives json formatted outline
```
2) Copy the o/p & paste it in Editor & Fill out JSON file with all the info for the Deployment Group & save it  as .json file
3) Upload the JSON file to create the new Deployment Group, which will include the SNS Trigger you added, by using the command:
 ```
 aws deploy create-deployment-group --cli-input-json file://<FILENAME>.json
 ```
b. Adding a Trigger to an existing Deployment Group:
1) Run the command:
```$aws deploy get-deployment-group --application-name TestApplication
--deployment-group-name TestDeploymentGroupName  /*gives json formatted o/p
```

2) Copy the entire JSON o/p text block
3) Create a .json file and open it with a text editor
```sh
$touch CLITriggerFile.json
$nano CLITriggerFile.json   /*paste the copied JSON formatted o/p
```
4) Remove the following items from the copied JSON file
- "deploymentGroupInfo": {
- "deploymentGroupId": “XXXX”,
- "deploymentGroupName": ”XXXX”,
- The entire “targetRevision” section (if your file has it)
- The remaining “}” from when you deleted "deploymentGroupInfo” above
5) Add the following JSON text at triggerConfigurations section & save it.
```sh
 "triggerConfigurations": [
 {
    "triggerEvents": [
      “DeploymentStart”,
      ”DeploymentFailure”
                      ],
    "triggerTargetArn": <“copy Arn from AWS Console of SNS Topic for the "Trigger Topic" which we subscribed”>,
    "triggerName": “CLICreatedTrigger"
 }
],
```
6) Upload the .json file using the command:
```sh
$aws deploy update-deployment-group --current-deployment-group-name TestDeploymentGroupName --cli-input-json file://CLITriggerFile.json
/*gives “hooksNotCleanedUp” as o/p
```
c. Viewing, Modifying & Deleting a Trigger in an existing Deployment Group:
1) To view the triggers in a Deployment Group, run:
```sh
$aws deploy get-deployment-group --application-name TestApplication --deployment-group-name TestDeploymentGroupName /*gives JSON formatted o/p including newly added TriggerArn
```
2) To modify or delete triggers:
    - Open the .json file:
        - Add events , Remove events , Delete the entire contents of the “trigger configuration section”
3) Save and exit
4) Re-upload the .json file
##### j.Viewing Deployment Details and Error Logs
##### Viewing Deployment Details via AWS CONSOLE:
1. Deployment details:
AWS—>CodeDeploy—>Click on Dropdown menu of AWS Codedeploy at the top—>select Deployments—>lists deployments & its details
2. Instance details:
AWS—>CodeDeploy—>Click on Dropdown menu of AWS Codedeploy at the top—> select Deployments—>Deployments list—>select any one Deployment details->View All Instances—>View Events—>instance Failed/succeeded—>select Failed event “view logs”—>to view error to troubleshoot
3. Application details:
AWS—>CodeDeploy—>Click on Dropdown menu of AWS Codedeploy at the top—> select Applications—>lists Applications & its details
4. Deployment Group details:
AWS—>CodeDeploy—>Click on Dropdown menu of AWS Codedeploy at the top—> select Applications—>lists Applications—>click on Application—> Deployment Group—>Details(click on Downward arrow ‘v’ beside deployment group)
5. Application Revision details:
AWS—>CodeDeploy—>Click on Dropdown menu of AWS Codedeploy at the top—> select Applications—>lists Applications—>click on Application-> Revisions—>gives detailed info
6. Deployment Configuration details:
AWS—>CodeDeploy—>Click on Dropdown menu of AWS Codedeploy at the top—> select Deployment Configuration
##### Viewing Deployment Details via CLI:
1. Deployment details:
  - List Deployment ID numbers:
```sh   
aws deploy list-deployments --application-name <APP-NAME> --deployment-group-name <DEPLOY-GROUP-NAME>
```
  - Optional:
```sh
--include-only-statuses <Failed Or Succeeded> --create-time-range start=<2014-08-19T00:00:00>,end=<2014-08-20T00:00:00>
```
  - List detailed deployment info:
```aws deploy get-deployment --deployment-id <ID #>```
2. Instance details:
  - List instances
```aws deploy list-deployment-instances --deployment-id <ID #>```
  - List detailed information about an instance
```sh
aws deploy get-deployment-instance --deployment-id <DEPLOYMENT ID> --instance-id <INSTANCE ID>
```
3. Application details:
  - List Applications:
```aws deploy list-applications```  
  - List detailed Application info:
```aws deploy get-application --application-name  <APP-NAME>```
4. Deployment Group details:
  - List Deployment Groups:
```aws deploy list-deployment-groups --application-name <APP-NAME>```
  - List detailed Deployment Group info:
```sh
aws deploy get-deployment-group --application-name <APP-NAME> --deployment-group-name <DEPLOYMENT-GROUP-NAME>
```
5. Application Revision details:
  - List Application Revisions:
```aws deploy list-application-revisions --application-name <APP-NAME>```
  - List detailed Application Revision info:
```sh
aws deploy get-application-revision --application-name <APP-NAME> --s3-location bucket=<S3-BUCKET-NAME>, bundleType=<TYPE>,eTag=<TAG-VALUE>,key=<FILE-NAME>
```
6. Deployment Configuration details:
  - List Deployment Configurations:
```aws deploy list-deployment-configs```
  - List detailed Deployment Configuration info:
```sh
aws deploy get-deployment-config --deployment-config-name <CONFIG-NAME>
```
##### k.Stopping Deployments, Roll-Backs, and Redeployments
##### Stopping a Deployment:
- AWS Console:
1) While a deployment is deploying, be on the “Deployments” page in CodeDeploy.
- AWS—>CodeDeploy—>Click on Dropdown menu of AWS Codedeploy at the top—>Create New Deployment—> Application—>”TestApplication”—> Deployment Group—>”TestDeploymentGroupName”—>Revision Type—>select “My application is stored in Amazon S3”—>Revision location—>select Revision location”—>Deployment description—>”STOP”—>Deployment Config—>Select “default one at a time”—>Deploy Now—>Actions—>Click on Stop button
- AWS CLI:
1) While a deployment is deploying, run the command:
```stop-deployment --deployment-id <DEPLOYMENT-ID #>```
- State of our Instances?
  - Partial or full file installation, Partial or all scripts run, Nothing run or installed
##### Rolling-Back & Redeploying:
Understanding Roll-Back and Redeployments:
1) CodeDeploy treats a “Redeployment” as just a deployment of an already deployed Revision.
2) CodeDeploy does not have an automatic “Roll-back” feature.
- “Cleanup” file:
    - The cleanup files is where CodeDeploy stores information about the last installed files,
    - File location (Linux):
    ```sh
    /opt/codedeploy-agent/deployment-root/deployment instructions/<DEPLOYMENT-GROUP-ID>-cleanup
    ```
- login to EC2 instance of deployed instance
```sh
[ec2@…]$cd /opt/codedeploy-agent/deployment-root/deployment instructions/
[ec2@…]$ls   /*lists cleanup files including installed files
[ec2@…]$nano  <copy & paste one of the cleanup file>  /*lists of all files that are in our deployment.
```
- so that in the case of redeployment, CodeDeploy can try to delete these files,then install files that are redeployed in our Revision.
- File location (Windows):
```sh
C:\ProgramData\Amazon\CodeDeploy\deployment instructions\<DEPLOYMENT-GROUP-ID>-cleanup
```
##### l. Automating Deployments from S3 Using Lambda
Process overview:
1) Create a Lambda “execution role”: Give Lambda permission to access S3 & CodeDeploy
2) Configure a Lambda Function:  Prepare the code for the Lambda Function
3) Register a Lambda Function: Setup the Lambda Function in AWS
4) Bundle the source files into a .zip file:
5) Upload the .zip file the an S3 bucket:
##### Creating the Lambda “Execution Role”:
1) Create a custom IAM policy:
- AWS—>IAM—>Policies—>Create New Policy—>Create our own Policy—>Policy Name “LambdaExecutionPolicy”->Policy Document—>[Copy & Paste “CodeDeployLambdaExecutionPolicy” which is in git files—>Enter the appropriate  <Bucket Name>,<Region>,<Account-ID> & ID will be in our AWS-Settings-"My Account”] —> Validate Policy—>Create Policy(if its Valid).
2) Create a new Role:
- AWS—>IAM—>Roles—>Create New Role—>”LambdaExecutionRole”—>Role Type—>select “AWS Lambda”—>Attach Policy—>select “LambdaExecutionPolicy”(which we created above)—>create
##### Creating the Lambda Function:
1) Get the Lambda Function code:
2) Create a Lambda Function:
- AWS—>Lambda—>Create a Lambda Function—>Click “Skip”—>Name “CodeDeployLambdaAutomation" and [copy the script into the “code” section from  “CodeDeployLambdaAutomation” file]—>Lambda function handler & Role—>Role—>select “LambdaExecutionRole”—>Next—>Create function—>click Event Source/Triggers Tab—>Add an Event Source—>Select Source Type “S3"
 —>Select Bucket “Appropriate target S3 Bucket”—>Set Event Type = Object Created (All)—>select “Enabled event source”—>“Submit”
##### Preparing and Uploading the Source Files:
1) Creating a .zip file of the source files:  
- Make sure our appspec.yml file is moved outside our root application folder (should be one directory up)
- Goto CLI,
```sh
$ls  /* we need to view “apsec.yml” file under our User1 account,if not move it by using move command.
```
- Install zip if we don’t already have it installed
- Run the following command to properly create the zip file:
```sh
zip -r <NAME-OF-ZIP-FILE>.zip appspec.yml <APP-DIRECTORY> -x "*/\.*”
$zip -r  AutoRevision.zip  appspec.yml  local-wonderwidgets  -x  "*/\.*”  /* lists allies & folders that included in zip file [ here ‘x’ means exclude any hidden files/folders that meets these(/,\,.) parameters]
$ls /* including AutoRevision.zip files will be listed
```
2) Upload the zip file to S3:
- Run the S3 PUT command:
```sh
- aws s3api put-object --bucket <S3-BUCKET-NAME> --key <ZIP-FILE-NAME> .zip --body <ZIP-FILE-NAME> .zip --metadata application-name=<APP-NAME>, deploymentgroup-name=<DEPLOYMENT-GROUP-NAME>
- $aws s3api put-object --bucket wonder widgets-deployment-test --key AutoRevision.zip --body AutoRevision.zip --metadata application-name=TestApplication, deploymentgroup-name=TestDeploymentGroupName
/*gives ETag as o/p
```
- To check whether successfully deployed revision or not goto
- AWS—>CodeDeploy—>AWSCodeDeploy—>Deployments—>TestApplication-> status—>succeeded
#### CodeDeploy Basic example:
- AWS—>IAM—>Create New Role—>Name->CodeDeployInstanceRole->create
- AWS—>EC2—>Launch an Instance—>AMI—>Free Tier—>Configure Instance—>Auto-assign Public IP—>select Enable—>IAM Role—>select CodeDeployInstanceRole—>Advanced Details—>Copy & paste the following bash script—>Add Storage—>Tag Instance—>Name “LiveLabCodeDeployInstance”—>Configure SG—>Add—>SSH & HTTP “Anywhere”—>launch—>create a new keypair—>”LiveLabKeyPair”—> Download—>launch instance
```sh  
#! /bin/bash
sudo yum update
sudo yum install ruby
sudo yum install wget
cd /home/ec2-user
wget https://aws-codedeploy-us-east-1.s3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
```
- connect to running “LiveLabCodeDeployInstance” through ssh
- goto CLI,
```sh
$cd downloads  /*cd to dir where key pair was downloaded
$chmod 400 LiveLabKeyPair.pem
$ssh -i "LiveLabKeyPair.pem” <public-ip of ec2>
$sudo service codedeploy-agent status  /*to check the code-deploy status
```
#### CodePipeline
- CodePipeline is a continuous delivery service. It provides the tools to model, visualize, and automate the many steps that are required as part of the software release process.
- Automation : We can easily automate the entire release process, from the source repository all the way to production servers.
- Consistency :  Create and repeat a consistent set of steps each time you want to release or update your software.
- Speed up the delivery process :  Speed up the release process through automation.
- Use existing tools : CodePipeline integrates with many other AWS services (CodeCommit, CodeDeploy, S3, OpsWorks), as well as other major developer and DevOps platforms such as Jenkins and GitHub.
- Easy to visualize and view :  See each stage of the release process, their status, and position.
##### Software Release Process:
Source Files-—>Build-—>Deployment-—>Testing Environment-—>Deployment-->Production Environment. Including Customer Feedback & Developer Updates
- Source Files: CodeCommit,S3,Github
- Build: Jenkins
- Deployment: CodeDeploy
- Testing Environment: Testers,EC2,Apica,Solano
- Production Environment: EC2, Customers

Continuous Delivery: A software engineering approach where teams produce software in short cycles, ensuring that the software can be reliably released at any time. It aims for building, testing, and releasing software faster and more frequently. The approach helps reduce the cost, time, and risk of delivering changes by allowing for more incremental updates to applications in production. ```A straightforward and repeatable deployment process is important for continuous delivery.```

Continuous Integration: The practice of merging all developer working copies to a shared mainline (such as a master branch), at a continuous pace (often several times a day). Each addition (change) is built and tested as quickly as possible.

##### AWS CodePipeline Concepts & Terminology:
1. Pipeline:
- A workflow framework that helps you create and manage the release process. (blueprint)
- It is how you specify, build, coordinate, monitor, and execute your specific release process.
- Consists of:
  - Stages (which consist of Actions)
  - Transitions (between each Stage)
- The first time you create a pipeline (AWS Console) an S3 bucket is automatically created that will store the pipeline’s artifacts:
  - Created in the same region as your pipeline
  - The bucket is named “codepipeline-<REGION>-<RANDOM_10_DIGIT_#>”
- We can have up to 20 pipelines per AWS account
- Pipelines must be in one of the following regions:
  - us-east-1 , us-west-2 , eu-west-1
2. Stages:
- Each pipeline is broken up into broad sections called ```stages```.
- Stages are used to categorize, execute, and monitor actions.
- Stages are completed in order – as configured in the pipeline.
- Every stage must have at least one action.
- AWS default stages include:
  - Source,Build,Beta
- A stage can only process one ```revision``` at a time.
- Stages are connection through ```transitions```.
3. Actions:
- An action is a task performed on the ```artifact```.
- Action tasks include:
  - Source: Monitors the “source” location (i.e. CodeCommit, S3, GitHub) for new commits or uploaded revision – and starts the release process if found
  - Build: Code is built (i.e. Jenkins)
  - Test: Run the code through a test provider (i.e. BlazeMeter, Apica, etc.)  
  - Deploy: Install the application files onto a fleet of instances
  - Invoke: Trigger a Lambda function
  - Approve: Require human approval before moving to the next stage

Note: Services, such as CodeCommit, Github, Jenkins, Apica, CodeDeploy, etc – are referred to as action “providers”

4.Revisions, Artifacts & Transitions:
- ```Revision```: is the general term used to describe the code “update” or version that is currently running through the pipeline.
- ```Artifacts```: refer to the actual set of files (objects) that are being passed through the pipeline, and is categorized by input or output artifacts.
- For example: The un-built source files being passed into the “build” stage are the “input artifacts.” The built files (after running through Jenkins), are the “output artifacts.”
- The “output artifacts” of a stage are then passed to the next stage via transitions.
- Artifacts are stored in the S3 bucket that was created or designated when you create a pipeline.
- ```Transitions```: tell the artifact what stage to go to next, and act as a delivery system between them.
- Transitions can be enabled or disabled to allow or prevent upcoming stages to be run.
