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
#### 2.Cloning Repositories, Commits, Push, and Pulls
$aws codecommit  list-repositories
Assume that there are 3 users, User1 & User2 & User3
-To clone a repo:
AWS—Code Commit—select repo name “Myrepo" — code—clone URL— select SSH/HTTP url
-In CLI,
-Here “Myrepo” is the central repo
-Here User1 making changes to “Myrepo” by cloning into his local repo
$git clone <url> <Myreponame> <My-local-repo-name>
$ls  /*to check local repo cloned/not
$cd <My-local-repo-name>
$ls   /*lists files which were created in “Myrepo”, which cloned now
-Now make some changes in local repo by creating a file “what_is_www”
$touch what_is_www
$nano what_is_www   /*to edit file
#!/bin/bash
#My first script
echo “hello"
save & exit
$chmod 755 what_is_www  /*changing permissions to execute
$./what_is_www
hello
$git status  /*tells us that there are untracked files “what_is_www”
$git add what_is_www   /*to add file to commit
$git status     /*gives changes committed
$git rm - -cached what_is_www    /*to remove file, if it added by mistake
$git status  /*again gives, untracked files
$git add what_is_www   /*so again add it if we want that file & check status
$git commit -m “adding file to central repo”   /*asks for email & name if not configured
-To configure user globally
$git config  - -global  user.email  “username@Myreponame”
$git config  - -global  user.name  “username”
$git commit -m “adding file to central repo”   /*Now it commit changes
$git remote    /* to get remote name
$git branch    /*to get branch name
$git diff  - -stat  <remote name>  <branch name>   /*to view which files to be pushed
$git push <remote name>/<branch name>  
-After push success, check file pushed to  “MyRepo” or not
-Goto AWS — Code Commit — select repo — “MyRepo” — “what_is_www”.

-Now User2 has “Myrepo” with .txt files but there is no “what_is_www” file in it.
-So User2 has to clone the “Myrepo” with new changes i.e.,“what_is_www” file
$ls    /*lists “User2-local-repo” dir , which already cloned from “Myrepo” to local repo  before  changes pushed by User1
$cd  User2-local-repo
$ls    /*files will be listed
-Now User2 has to Pull the new changes from “Myrepo” which are pushed by User1
$git remote
$git branch
$git pull <remote name>  <branch name>
$ls   /*lists what_is_www file along with previous files
$./what_is_www    /*gives info which is in file
Now User1 & User2 both have central repo with same files.
