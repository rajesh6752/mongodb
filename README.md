## Set up MongoDb

### Install MongoDB

Step 1: Update Amazon Linux 2 Server:
```
sudo yum -y update
```
Step 2: Add MongoDB YUM Repositories:
#### Create a /etc/yum.repos.d/mongodb-org-5.0.repo file so that you can install MongoDB directly using yum:
```
sudo tee /etc/yum.repos.d/mongodb-org-5.0.repo<<EOL
[mongodb-org-5.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2/mongodb-org/5.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-5.0.asc
EOL
```
Confirm the repository is now available:

```
sudo yum repolist
```
