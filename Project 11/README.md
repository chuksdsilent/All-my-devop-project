# DEPLOY APP USING K8

1. Create a k8 cluster using K0PS with the following commands
```
kops create cluster --name=kims.oshabz.name.ng \ 
--state=s3://kops-k8-2022 --yes  --zones=us-east-1a,us-east-1b \ 
--node-count=2 --node-size=t3.small --master-size=t3.medium --dns-zone=kims.oshabz.name.ng \ 
--node-volume-size=8 --master-volume-size=8


kops update cluster --name kims.oshabz.name.ng --state=s3://kops-k8-2022 --yes 


kops validate cluster --name kims.oshabz.name.ng --state=s3://kops-k8-2022

kops delete cluster --name kims.oshabz.name.ng --state=s3://kops-k8-2022 --yes 

```

2. Create a volume
```
aws ec2 create-volume --availability-zone=us-east-1b --size=3 --volume-type=gp2
```

3. label your nodes
```
kubectl get nodes
kubectl node node-name | grep us-east-1
kubectl label nodes node-name zone=us-east-1a
```

4. Add tag to the volume created
