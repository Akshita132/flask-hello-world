
[Documentation_Akshita Jain.pdf](https://github.com/user-attachments/files/16398775/Documentation_Akshita.Jain.pdf)
Problem Statement

	Web Application Deployment on Azure Cloud

Task 1: Set Up a VNet and Ubuntu VM on Azure
1.1	Create a Virtual Network (VNet)
1.	Navigate to Azure Portal: Open the Azure Portal at portal.azure.com.
2.	Create a VNet:
o	Search for "Virtual Networks" in the search bar.
o	Click on "Create" and fill in the required details:
	Resource group: Akshita-RG
	Name: Akshita-VM-vnet
	Address space: 10.0.0.0/16
	Subnet name: default
	Subnet address range: 10.0.0.0/24
o	Click "Review + create" and then "Create".



1.2	Create an Ubuntu VM
1.	Create a VM:
o	Search for "Virtual Machines" in the search bar.
o	Click on "Create" and select "Virtual machine".
o	Fill in the required details:
	Resource Group: Choose the one which we created while creating VNet (Akshita-RG).
	Virtual machine name: Akshita-VM
	Region: Central US
	Image: Ubuntu Server 22.04 LTS
	Size: Standard D2s v3 (2 vcpus, 8GiB memory)
	Authentication type: SSH public key
 
	Username: azureuser
	SSH public key: generate a new key pair
2.	Under "Networking", select the VNet that has been created (Akshita-VM-vnet) and the default subnet.
3.	Ensure "Public IP" is set to "Enabled".
4.	Under "Inbound port rules", allow HTTP (port 80) & SSH (port 22).
5.	Click "Review + create" and then "Create".




1.3	Connect to the VM Using SSH
1.	Obtain the VM's Public IP:
o	Navigate to the "Overview" section of the VM in the Azure Portal.
o	Copy the public IP address.
2.	SSH into the VM:

•	ssh -i <pem file> <username>@<public-ip-address>
•	ssh -i Akshita-VM_key.pem azureuser@52.180.144.2

1.4	Host a Basic HTML Page
1.	Install Nginx:

sudo apt update
sudo apt install Nginx -y
 
 

 
 

2.	Create a Basic HTML Page:
•	Go to /var/www/html/
•	Create a file index.html and mentioned the below code.


 
 
3.	Verify the Page:
1.	Open a web browser and navigate to 52.180.144.2 (public ip address)
2.	We should see the "Welcome Akshita!" message.



Task 2: Create an Azure Storage Account and Upload HTML Page
2.1	Create a Storage Account
1.	Create a Storage Account:
o	Search for "Storage accounts" in the search bar.
o	Click on "Create" and fill in the required details:
	Resource Group: Use the same resource group as the VM (Akshita-RG).
	Storage account name: akshitasan
	Region: Select the same region as the VM (Central US).
 
o	Click "Review + create" and then "Create".



2.2	Configure and Upload HTML File
1.	Install Azure CLI on the VM:

•	sudo apt update

•	sudo apt install azure-cli -y


2.	Login to Azure:

•	az login
 
 

3.	Create a Storage Container:

Create a storage container with name akshita-html as shown below.


Go to Access Control (IAM) and add Storage Blob Data Contributor role to be able to upload file from VM to container.


4.	Upload the HTML File:

az storage blob upload --account-name akshitasan
 
--container-name akshita-html --name index.html --file /var/www/html/index.html –auth-mode login


5.	Verify the Upload:
o	Go to the Azure Portal, navigate to the storage account, and check the "akshita-html" container.
o	Ensure the index.html file is present.

•	Copy the url of index.html file from container and paste it on the browser.
•	We should see “Welcome Akshita!”.

Task 3: Fork and Deploy a Flask Application
3.1	Fork the Repository
1.	Fork the Repository:
o	Navigate to flask-hello-world repository.
o	Click on "Fork" to create a copy of the repository.
3.2	Deploy the Application on the VM
1.	SSH into the VM:

•	ssh -i Akshita-VM_key.pem azureuser@52.180.144.2
 
2.	Install Python and Git:

sudo apt update
sudo apt install python3 python3-pip git -y

3.	Clone the Forked Repository:

git clone https://github.com/Akshta132/flask-hello-world.git cd flask-hello-world

4.	Install Flask and Gunicorn:

pip3 install -r requirements.txt

5.	Run the Application:

python3 app.py and make sure to open port 5000 in inbound rules of VM. Then browse 52.180.144.2:5000. I have also made below changes in code to be able to run it.

from flask import Flask app = Flask(  name  )
@app.route('/') def hello_world():
return 'Hello, World!'
if  name  == ' main ': app.run(host='0.0.0.0', port=5000)




 
 


6.	Create a Systemd Service:
o	Create a new service file:
sudo nano /etc/systemd/system/ flask-hello-world.service

o	Add the following content to the file:
[Unit]
Description=Flask Hello World After = network.target
[Service] User=azureuser
WorkingDirectory=/home/azureuser/flask-hello-world ExecStart=/usr/bin/python3 /home/azureuser/flask-hello-world/app.py Restart=always
[Install]
WantedBy=multi-user.target


o	Enable and start the service:
sudo systemctl enable flask-hello-world sudo systemctl start flask-hello-world


7.	Verify the Application:
o	Stop and start the VM and then browse 52.180.144.2:5000 to test the application
o	We can see "Hello World!"

 
Task 4: Automate Deployment with GitHub Actions
4.1	Modify app.py to Display Your Name
1.	Edit app.py:

vi app.py

2.	Modify the Content:

from flask import Flask app = Flask(  name  )
@app.route('/') def hello_world():
return 'Hello Akshita!'

if  name  == ' main ': app.run(host='0.0.0.0', port=5000)

3.	Commit and Push the Changes:

git add app.py
git commit -m "Update app.py to display my name" git push origin master

4.2	Create a GitHub Actions Workflow
1.	Generate an SSH Key Pair:

ssh-keygen -t rsa -b 4096 -C "github-actions@Akshita-vm"

o	Follow the prompts to save the key in the default location and leave the passphrase empty.
2.	Add the Public Key to GitHub:
o	Copy the public key:
cat ~/.ssh/id_rsa.pub

o	Add it to the Personal Access Tocken on GitHub.
3.	Add the Private Key to GitHub Secrets:
o	Copy the pem file that we use to connect to VM:
cat Akshia-VM_key.pem

o	Add it to your repository's "Secrets" as AKSHITA_AZURE_PEM.
4.	Create a Workflow File:
o	In the repository, create a directory called .github/workflows.
o	Inside this directory, create a file named deploy-workflow.yml.
 
5.	Add the Workflow Configuration:


6.	Commit and Push the Workflow File:

git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions workflow for deployment" git push origin master

4.3	Verify and Run the Workflow
1.	Trigger the Workflow:
o	Make a commit to the master branch to trigger the workflow automatically.



 
 

2.	Verify the Deployment:
o	After the workflow runs, verify that the Flask application is updated and running on the VM by accessing http:// 52.180.144.2:5000 in a web browser.
o	Made change in app.py after creating action. It is now working as expected:



o	We can now see "Hello Akshita Jain! How are you?".

Task 5
1.	What are the steps to set up a VNet and Ubuntu VM in Azure, and how do you configure inbound ports for HTTP access?

Sol:

•	Create a VNet in Azure with a specified address space and subnet.
•	Create an Ubuntu VM within this VNet, allowing HTTP (port 80) and SSH (port 22) inbound traffic.
•	Connect to the VM using SSH with the public IP address.
•	Install Nginx and host a basic HTML page to verify HTTP access.
 
2.	How do you configure an Azure storage account and upload files to it using an SSH connection from your VM?

Sol:

•	Create a storage account in Azure and a container with public access.
•	Install Azure CLI on the VM and log in to your Azure account.
•	Use Azure CLI commands to upload files from the VM to the storage container.

3.	Explain the process of creating a GitHub workflow to automate deployment from GitHub to an Azure VM. What are the key considerations to ensure it functions correctly?

Sol:

•	Generate an SSH key pair and add the public key to GitHub and the private key to GitHub Secrets.
•	Create a GitHub Actions workflow file to set up SSH and deploy the code to the VM.
•	Trigger the workflow on code push or manually to pull the latest code from GitHub and restart the application on the VM.
•	Ensure the SSH keys, VM IP, and permissions are correctly configured for successful deployment.

Challenges:

•	Managing permissions and access control for the Azure storage account to securely upload files.
•	Writing and debugging the GitHub Actions workflow file to ensure it correctly deploys the application.
Thank you!

