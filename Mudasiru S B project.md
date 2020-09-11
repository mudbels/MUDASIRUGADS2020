# LAB: AK8S-04 Creating a GKE Cluster via Cloud Shell

## Objectives:

  In this lab, you learn how to perform the following tasks:

 - Use kubectl to build and manipulate GKE clusters.
 
 - Use kubectl and configuration files to deploy Pods.

- Use Container Registry to store and deploy containers.

## Steps:

**1. Deploy GKE clusters**

 - In Cloud Shell, type the following command to set the environment   variable for the zone and cluster name.

    export my_zone=us-central1-a
    export my_cluster=standard-cluster-1



 - In Cloud Shell, type the following command to create a Kubernetes cluster.

    gcloud container clusters create $my_cluster --num-nodes 3 --zone $my_zone --enable-ip-alias

**2.  Modify GKE clusters**

 - In Cloud Shell, execute the following command to modify standard-cluster-1 to have four nodes.
   
    gcloud container clusters resize $my_cluster --zone $my_zone --size=4

 - press y to confirm.

**3. Connect to a GKE cluster**

 - To create a kubeconfig file with the credentials of the current user (to allow authentication) and provide the endpoint details for a specific cluster (to allow communicating with that cluster through the kubectl command-line tool), execute the following command.

    gcloud container clusters get-credentials $my_cluster --zone $my_zone

 - Open the kubeconfig file with the nano text editor
 
    nano ~/.kube/config

 - Press CTRL+X to exit the nano editor.

**4. Use kubectl to inspect a GKE cluster**

  - In Cloud Shell, execute the following command to print out the content of the kubeconfig file

    kubectl config view

 - In Cloud Shell, execute the following command to print out the cluster information for the active context

    kubectl cluster-info

 - In Cloud Shell, execute the following command to print out the active context.

    kubectl config current-context
 - print out some details for all the cluster contexts in the kubeconfig file:
 
    kubectl config get-contexts

 -  change the active context
 
     kubectl config use-context gke_${GOOGLE_CLOUD_PROJECT}_us-central1-a_standard-cluster-1

 - view the resource usage across the nodes of the cluster
 
    kubectl top nodes

  - enable bash autocompletion for kubectl:
  
    source <(kubectl completion bash)

  - type kubectl and press the Tab key twice

     - The shell outputs all the possible commands.
  
  -  type kubectl co and press the Tab key twice

     - The shell outputs all commands starting with "co" (or any other text you type)
**
5. Deploy Pods to GKE clusters**

   - deploy the latest version of nginx as a Pod named nginx-1:
   
     kubectl create deployment nginx-1 --image nginx:latest

 - view all the deployed Pods in the active context cluster:

    kubectl get pods

  - You will now enter your Pod name into a variable that we will use throughout this lab. Using variables like this can help you minimize human error when typing long names. You must type your Pod's unique name in place of [your_pod_name]

    export my_nginx_pod=[your_pod_name]

  - Confirm that you have set the environment variable successfully by having the shell echo the value back to you:

    echo $my_nginx_pod

  - view the complete details of the Pod you just created.
  
     kubectl describe pod $my_nginx_pod

#Push a file into a container

- open a file named test.html in the nano text editor.

    nano ~/test.html


- Add the following text (shell script) to the empty test.html file:
 
    <html> <header><title>This is title</title></header>
    <body> Hello world </body>
    </html>

       - Press CTRL+X, then press Y and enter to save the file and exit the nano editor.

- place the file into the appropriate location within the nginx container in the nginx Pod to be served statically:

    kubectl cp ~/test.html $my_nginx_pod:/usr/share/nginx/html/test.html

#Expose the Pod for testing

- create a service to expose our nginx Pod externally:

    kubectl expose pod $my_nginx_pod --port 80 --type LoadBalancer

- view details about services in the cluster:

    kubectl get services

- verify that the nginx container is serving the static HTML file that you copied,you replace [EXTERNAL_IP] with the external IP address of your service that you obtained from the output of the previous step;

    curl http://[EXTERNAL_IP]/test.html

- view the resources being used by the nginx Pod:

    kubectl top pods

**6.Introspect GKE Pods**

- clone the repository to the lab Cloud Shell.

    git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst

- Change to the directory that contains the sample files for this lab.

    cd ~/training-data-analyst/courses/ak8s/04_GKE_Shell/

- deploy your manifest:

    kubectl apply -f ./new-nginx-pod.yaml


- To see a list of Pods:

    kubectl get pods

#Use shell redirection to connect to a Pod


- start an interactive bash shell in the nginx container:

    kubectl exec -it new-nginx /bin/bash



- In Cloud Shell, in the nginx bash shell, execute the following commands to install the nano text editor:

    apt-get update

    apt-get install nano



- In Cloud Shell, in the nginx bash shell, execute the following commands to switch to the static files directory and create a test.html file:

    cd /usr/share/nginx/html

     nano test.html


- In Cloud Shell, in the nginx bash shell nano session, type the following text:

    <html> <header><title>This is title</title></header>
    <body> Hello world </body>
    </html>


   - Press CTRL+X, then press Y and enter to save the file and exit the nano editor.



-  exit the nginx bash shell:

    exit


- set up port forwarding from Cloud Shell to the nginx Pod (from port 10081 of the Cloud Shell VM to port 80 of the nginx container):

    kubectl port-forward new-nginx 10081:80



- test the modified nginx container through the port forwarding in another cloud shell tab:

    curl http://127.0.0.1:10081/test.html

  - The HTML text you placed in the test.html file is displayed
     <html> <header><title>This is title</title></header>
     <body> Hello world </body>
     </html>

#View the logs of a Pod

- in another new cloud shel tab,display the logs and to stream new logs as they arrive (and also include timestamps) for the new-nginx Pod:

    kubectl logs new-nginx -f --timestamps

- You will see the logs display in this new window


- Review the additional log messages as they appear in the third Cloud Shell window.


- Close the third Cloud Shell window to stop displaying the log messages.
- Close the original Cloud Shell window to stop the port forwarding process.