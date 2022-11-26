1. Clone the repo, and checkout to a new branch, (say) dev. 

2. Create a .gitignore file, and add starter/terraform/.terraform. The binaries created by terraform are too large to be
pushed to a remote repo. 

###### PROJECT BEGINNING ######

3. Create two S3 buckets in us-east-1. One will contain the AMI Image in us-east-1. The second bucket is the one terraform will use.
``` aws ec2 create-restore-image-task --object-key ami-08dff635fabae32e7.bin --bucket udacity-srend --name "udacity-<ami_bucket_name>" ```

4. The command above will send back an AMI image. Make note of this ID.

5. Use ``` aws ec2 copy-image --source-image-id <your-ami-id-from-above> --source-region us-east-1 --region us-east-2 --name "udacity-<your_name>" ```
to create a copy of the above ID in us-east02.

6. Go to us-east-2 in the console, and create a private key pair named udacity. This is required to create and ssh into the EC2 instance. 

7. Open the _config.tf and change the bucket name and region to the s3 bucket that you have created in a previous step.

8. Further replace the /starter/terraform/modules/ec2/ec2.tf to the following file: https://github.com/udacity/cd1898-Observing-Cloud-Resources/blob/503ffd8cc9b1c2fc8dda31beacae8d5b5380883f/starter/terraform/modules/ec2/ec2.tf

9. Run terraform init, and terraform apply

10. Check if the EKS stack and EC2 instance with username ubuntu have been created. (Check in us-east-2)

11. SSH into the EC2 instance using ``` ssh -i ssh -i /path/key-pair-name.pem instance-user-name@instance-public-dns-name ```

12. Install node_exporter using ``` sudo useradd --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.2.2.linux-amd64.tar.gz
sudo cp node_exporter-1.2.2.linux-amd64/node_exporter /usr/local/bin
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
sudo nano /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter

sudo ufw allow 9100/tcp
sudo systemctl restart ufw ```

13. Open Postman and import the environment and collection files from starter/

14. Before executing commands in EKS, the ```kubeconfig``` file needs to be updated.

Do: ```aws eks --region us-east-2  update-kubeconfig --name udacity-cluster```

15. Create a namespace called monitoring
``` kubectl create namespace monitoring ```

16. Create the ```prometheus-additional.yaml``` file. This will be used later to install `blackbox-exporter` in the kubernetes cluster. Use https://github.com/prometheus-operator/prometheus-operator/blob/main/example/additional-scrape-configs/prometheus-additional.yaml. 

17. In this file, add an additional target to the EC2 instance, node-exporter. 

18. Obtain the latest ```values.yaml``` file from https://raw.githubusercontent.com/prometheus-community/helm-charts/main/charts/kube-prometheus-stack/values.yaml.

Modify `values.yaml` near line 2310 so that it matches:
```additionalScrapeConfigsSecret:
enabled: true
name: additional-scrape-configs
key: prometheus-additional.yaml```.

19. Create a kubernetes secret which references the `prometheus-additional.yaml` file. 

```kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml --namespace monitoring```. Do not install black box exporter just yet. 

20. Add the prometheus repo to helm, and update.
`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update`.

21. Install the monitoring stack in the Kubernetes cluster using helm. 
`helm install prometheus prometheus-community/kube-prometheus-stack -f "starter/values.yaml" --namespace monitoring`

22. Open Lens, add the cluster, enable the monitoring namespace, and check if prometheus and grafana have been deployed. 

23. Log into Grafana using the credentials 
`user: admin & password: prom-operator`

24. Inside postman, in the environment tab - add the public-ip as the public ip of the EC2 server, and the username as ubuntu. 

