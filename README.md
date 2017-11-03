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
    - Mac OSX: SSH (due to keychain issues)

### Setup & Configuration
1.Configure IAM Users, Access Keys & Policies
-Create a IAM User(as CodeCommit),download key credentials of that user.
-Attach AWSCodeCommitFullAccess Policy to the user.
