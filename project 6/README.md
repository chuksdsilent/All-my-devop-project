# KUBERNETICS

## SETTING UP KOPS

1. Create an EC2 instance

2. Create a new user with programmatic access

3. Create s3 bucket

4. install awscli  with the following commands

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```

5. install kubectl from k8 website
```
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
```
6. Execute the following commands
```
chmod +x kubectl
sudo mv kubectl /usr/local/bin
kubectl --help
```

7. Install KOPS
```
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/kops
kops --help    
```

6. Login to aws >> Route53 >> Hosted Zone

