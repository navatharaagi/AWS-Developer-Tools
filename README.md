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
-credentials last for 15mins only, its issue in OSX(Mac).So git clone ‘macerrorrepo’ to our local repo.
mac$ git clone <url of macerror repo> <local repo name>
$cd local repo
$touch test.txt
$git add text.txt
$git commit -m “test commit”
$git push
-After 15 mins, if we do git pull, will get error.
-Now just goto “ keychain Access” in our sys,search for ‘ git’, it will show git keys, select the key which associated with macerrorrepo—Access control--select 'git-credential-osxkeychain’ —del—save—enter sys pwd.
$git pull   (Deny the given notification)  /*will get already up to date
