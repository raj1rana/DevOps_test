# Deployment stragety 


objective:- 
1.  - new git repo for each Microservice
    - These files must be present in all the repos
        -- Kubernetes Deployment Defination file 
        -- kubernetes service files
        -- Dockerfile
        -- code files
        -- Packer code (for example in case of jenkins Jenkinsfile)

2.  - CI/CD In case of Jenkins
    - Write a Jenkinsfile that is stored in repo of each microservice
    - configure your git repo to use Webhook so that it can track each push or merging of pull request automatically and pushes to the defined branches
    - Jenkins file should run a Dockerfile that will also be supplied by git repo 
    - Jenkis enviroment variables should contain all the creadentials related to the microservice so that it can be injected to the build 
    - Test cases should be written by the developer and an entry point file (eg phpunit.xml) must be present in the repo
    - To run the test cases Jenkins provide a varity of testing plugins and also we can write the code in the Jenkinsfile itself to execute that entry point file (eg phpunit.xml)
    - After the code is  tested it will execute the build step and the code will build if it needs building or compilation.
    - Jenkins will commit that Docker image and push it to container registry  with the tags same as ${branch name}+${build number}
    - A step which is named  update (Deployment-defination.yaml) will update the yam by shell ( sed -i) command to update the tag of the image (example ${branch name}+${build number}) so that the kubernetes can pull the latest tagged image
    - After this action we will trigger another build that will initiate the Kubectl command to re-run the deployment that will roll the new updated image with new tag with kubernetes Rolling-update principle
    - and the app will be deployed.

3.  - Deployment In Jenkinsfile will be based on the branched in git (eg;- if the branch is master the the deployment will go to live if the branch is stagig the code will go to staging and if the branch is dev the code will go to dev)
    - For UAT the Kubernetes cluster in case of AWS AKS will run on a 3 nodes cluster that will pull the imagename:masterlatest (ie ${branch name}+${build number} format)
    - After the UAT in production enviroment we can use 3 node cluster with Autoscaling enabled along with the kubernetes service type load balancer 
        -- For Production we can use Amazon API gateway to route traffic between different API's endpoints that is running in the AKS 
        -- For Database we can use RDS with MultiAZ enabled and autoscaling eabled that can scale the readers.
        -- For Redis we can use AWS ElasticCache that can provide an endpoint that is in same VPC as our AKS is in so that it can connect 
        -- For Long running Jobs we ca use AWS Lambda servers that can supports jobs running upto 30 mins
        -- For Queues we can use AWS SQS and SNS to make sure that the tracking of the queu is accurate everytime
        -- Public endpoints we can use Kubernetes service to make the Endpoints public and rest of the endpoints private
        -- User Interface we can use Kubernetes service to make the UI available to public with the domain and SSL configured in the LoadBalancer
        -- Internall API's must use a service that is working on the private network and must be available to public endpoints also with UI

    - For Staging we can create a new namespace  named 'staging' again that with Jenkinsfile kubectl command in which we will include only one node with the images pulling from imagename:staginglatest
    - For Dev we can create another new namespce names 'dev; again that with Jenkinsfile kubectl command in which we will include only one node with the images pulling from imagename:devlatest

4.  - For Logging we can use Kubernetes DemonSet for that to happen we need to convert our Daployment-defination.yaml into DemonSet-defination.yaml
    -  It runs in the background to perform a specific task. 'Fluentd' and 'filebeat' are two daemons that Kubernetes supports for log collection. It is imperative to set up a resource limit per daemon so that the collection of log files will be optimized according to the available system resources.

5. - for Alerts we can use AWS cloudwatch ad Cloudtrail metrix also we can use NewRelic if possible in production pods 
   

