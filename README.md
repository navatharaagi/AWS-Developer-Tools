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
#### 2.OSX: GIT Installation
- Download git for MacOSX
- To check whether git installed or not, run 'git help' command in CLI.
#### 3.Linux: GIT Installation
-Download git for linux or unix by following given commands according to distribution like CentOS(yum), ubuntu,…..from Google
#### 4.OSX/Linux: AWS CLI Installation
```
$python --version       /* to check whether python installed or not
#sudo yum install python3     /* to install python for CentOS
#sudo apt-get install python3    /* for  Debian distributions
#pip --help   /* to check pip installation
#curl -0 https://bootstrap.pypa.io/get-pip.py    /*to install pip
#sudo python3 get-pip.py
#sudo pip install awscli     /* to install AWS CLI
#sudo pip3 install awscli --ignore-installed six /*if we get error:six while AWS installation
#aws configure
AWS Access Key ID : Copy & paste access key of IAM user (code commit)
AWS Secret Key: Copy & paste secret key of IAM user (code commit)
region: enter region
format: json
Note: if we want to change the user change AWS access & secret keys by running 'aws configure’ command
