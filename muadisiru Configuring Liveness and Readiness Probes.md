##Configuring Liveness and Readiness Probes

#Objectives:


- In this lab, you learn how to perform the following tasks:

    Configure and test a liveness probe

    Configure and test a readiness probe

#steps

**1. Connect to the lab GKE Cluster**


 - set the environment variable for the zone and cluster name.

    export my_zone=us-central1-a
    export my_cluster=standard-cluster-1


 - Configure tab completion for the kubectl command-line tool.

    source <(kubectl completion bash)



 - Configure tab completion for the kubectl command-line tool.

    source <(kubectl completion bash)


 - Configure access to your cluster for kubectl:

    gcloud container clusters get-credentials $my_cluster --zone $my_zone


 - In Cloud Shell enter the following command to clone the lab repository to the lab Cloud Shell.

    git clone https://github.com/GoogleCloudPlatform/training-data-analyst



  - Create a soft link as a shortcut to the working directory.

    ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s



  - Change to the directory that contains the sample files for this lab.

    cd ~/ak8s/Probes/


**2. Configure liveness probes**




 - In the Cloud Shell, execute the following command to create the Pod:

    kubectl create -f exec-liveness.yaml



 - Within 30 seconds, view the Pod events:

    kubectl describe pod liveness-exec



 - After 35 seconds, view the Pod events again:

    kubectl describe pod liveness-exec


  - Wait another 60 seconds, and verify that the Container has been restarted:

    kubectl get pod liveness-exec


 - The output shows that RESTARTS has been incremented in response to the failure detected by the liveness probe



 - Delete the liveness probe demo pod.

    kubectl delete pod liveness-exec


**3. Configure readiness probes**


- Configure a Deployment with liveness and readiness probes


- create the readiness deployment:

    kubectl create -f readiness-deployment.yaml

 - Within 30 seconds, in the Cloud Shell, execute the following command to view the deployment events:

    kubectl describe deployment readiness-deployment

- Execute the following command in the Cloud Shell to view the status of the pods in your deployment:

    kubectl get pods

  - You should initially see 3 containers running with no restarts similar to this. For the first 30 seconds or so after they have started they will show that they are not ready


- Wait until 30 seconds have passed and execute the following command in the Cloud Shell to view the status of the pods in your deployment again:

    kubectl get pods

#Configure a load balancer Service



- In the Cloud Shell, execute the following command to create the load-balancer service:

    kubectl create -f readiness-service.yaml


- Check the status of the readiness-svc load balancer service

    kubectl describe service readiness-svc




 - Repeat the status check for the service until you see at least one active endpoint listed as shown here. You will probably see all three endpoints.

#Monitor the behavior of the liveness and readiness probes



- Now check the status of the pods again:

    kubectl get pods


- To check that you can connect to the application first query the external ip-address from the load balancer service details and save it in an environment variable:


    export EXTERNAL_IP=$(kubectl get services readiness-svc -o json | jq -r '.status.loadBalancer.ingress[0].ip')



- Check to see if the application is responding:

    curl $EXTERNAL_IP



  - Over the course of the next few minutes repeat the following commands every 10 seconds or so to check the deployment status and send a request to the load balancer.

    curl $EXTERNAL_IP

    kubectl get pods



 - You should continue to see responses with no failures or timeouts even though individual containers are being restarted regularly due to the failures detected by the liveness probes. If all three containers restart at more or less the same time you might still see a timeout but that should rarely happen.

The combination of the liveness and readiness probes provides a way to ensure that failed systems are restarted safely while the service only forwards traffic on to containers that are known to be able to respond.

End your lab

