---
page_type: sample
languages:
  - python
products:
  - azure
  - azure-redis-cache
description: "This sample creates a multi-container application in an Azure Kubernetes Service (AKS) cluster."
---

# Azure Voting App

This sample creates a multi-container application in an Azure Kubernetes Service (AKS) cluster. The application interface has been built using Python / Flask. The data component is using Redis.

To walk through a quick deployment of this application, see the AKS [quick start](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough?WT.mc_id=none-github-nepeters).

To walk through a complete experience where this code is packaged into container images, uploaded to Azure Container Registry, and then run in and AKS cluster, see the [AKS tutorials](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app?WT.mc_id=none-github-nepeters).

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

------------------------------------------------------------------

## A few step by step for quick reference
### Pre-requisites:
1. You already have an active Azure Subscription
2. You have a Github account and you perform basic operations
3. You have basic understanding of git operations.

1. Create a folder in your repo directory locally and clone the source repo
    
        git clone https://github.com/Bapic/azure-voting-app-redis.git
        cd azure-voting-app-redis
        docker-compose up -d
	
2. Run below to note the image ids, name of the docker images

        docker images
    
        docker ps

3. Launch http://localhost:8080 using a browser

4. Run below command to close the images

        docker-compose down
	
5. Login to Azure portal and copy the subscription id

6. Create a Resource group and set variables and create an AKS Cluster
	
	    $subscriptionId="your subscription id"
	    az login
	    az account set --subscription $subscriptionId
	
	
	    $name="voteAppAKSonAzure01"
	    $acrname="voteacr01"
	    $location="eastus"
	
	    az group create -l eastus -n $name
	
	    az aks create -n $name -g $name --node-count 1 --enable-addons 	monitoring --generate-ssh-keys
	    az aks get-credentials -n $name -g $name
	    set-content clusterinfo.txt (kubectl config get-contexts)
	    add-content clusterinfo.txt (kubectl get svc)
	    add-content clusterinfo.txt (kubectl get nodes)
	    add-content clusterinfo.txt (kubectl get pods)
	    add-content clusterinfo.txt (kubectl get ns --show-labels)
	    add-content clusterinfo.txt (kubectl -n cluster-config get deploy  -o wide)
	    add-content clusterinfo.txt (kubectl cluster-info)
	    code clusterinfo.txt

7. Create a ACR in the same resource group

	    az acr create --resource-group $name --name $acrname --sku Basic
	az acr login --name $acrname

8. Tag your local image

        docker tag azure-vote-front voteacr01.azurecr.io/azure-vote-front:v1

9. Push the images
	
        docker push voteacr01.azurecr.io/azure-vote-front:v1

10. List the image repo from ACR
	
        az acr repository list --name voteacr01.azurecr.io --output table

11. Update the all-in-one yaml file to include the acr name for the app version in the section 
	
        image: microsoft/azure-vote-front:v1

12. Deploy the app
	
        kubectl apply -f azure-vote-all-in-one-redis.yaml
        kubectl get service azure-vote-front --watch

13. Browse to test the application deployed using public ip of the AKS

14. Update the application locally to update the values in the config_file.cfg

15. Build and run the containers locally
	
        docker-compose up --build -d

16. Tag the new updated  image and push to acr
	
        docker tag azure-vote-front voteacr01.azurecr.io/azure-vote-front:v2
        docker push voteacr01.azurecr.io/azure-vote-front:v2

17. Check exising front-end pods and extend if required
	
        kubectl get pods
        kubectl scale --replicas=3 deployment/azure-vote-front

18. Update the application and monitor
	
        kubectl set image deployment azure-vote-front azure-vote-front=voteacr01.azurecr.io/azure-vote-front:v2
        kubectl get pods

19. Check the new udpated service by opening the public Ip of the AKS cluster

20. Scaling of the application
	..in progress...

Cluster upgrade
	        
          ..in progress...


**Ref & Notes:** https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app.

This will destroy all your images and containers. There is no way to restore them!
	    
      docker rm $(docker ps -a -q)
      docker rmi $(docker images -q)
In case you want to delete even those images that are referenced in repositories, use

    docker rmi $(docker images -q) --force