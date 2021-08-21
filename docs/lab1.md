Installing and Configuring Flux with GitHub
Introduction
This lab introduces the steps necessary for installing Flux and configuring it to work with a repository in GitHub. We'll need our own GitHub account to fork a sample repository, and this lab will spin up a Kubernetes cluster to enable us to install and configure Flux.

Create a GitHub Repository
The linuxacademy/content-gitops repository is already online for us. Let's get into it and press the Fork button. Once the new repository is created, we'll use the credentials on the hands-on lab overview page to log into our Kubernetes master node as cloud_user.

Deploy Flux Into Your Cluster
Let's make sure Kubernetes is running, and that we have some nodes:

$ kubectl get nodes
Now let's make sure Flux is installed and running:

$ fluxctl version
We should get response of unversioned, which is fine. Now let's take another look at our Kubernetes deployment:

$ kubectl get pods --all-namespaces
Create a namespace for Flux:

$ kubectl create namespace flux
This will let us make sure it got created:

$ kubectl get namespaces
Set the GHUSER environment variable, then check to make sure it was set:

$ export GHUSER=[Our GitHub Handle]
$ env | grep GH
Now we can deploy Flux, using the fluxctl command:

$ fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/content-gitops \
--git-path=namespaces,workloads \
--namespace=flux | kubectl apply -f -
Verify The Deployment and Obtain the RSA Key
Once that fluxctl command is finished running, we can verify:

$ kubectl get pods --all-namespaces
$ kubectl -n flux rollout status deployment/flux
Now we can get the Flux RSA key created by fluxctl:

$ fluxctl identity --k8s-fwd-ns flux
Copy that RSA key, and let's head back over to GitHub.

Implement the RSA Key in GitHub
In the GitHub user interface, make sure we're in our new repository and click on the Settings tab. In there, click Deploy keys, then click the Add deploy key button. We can give it a Title of something like GitOps Deploy Key, then paste the key we copied earlier down in the Key field. Check the Allow write access box, and then click Add key.

Use the fluxctl sync Command to Synchronize the Cluster with the Repository
Use fluxctl to sync the cluster with the new repository:

$ fluxctl sync --k8s-fwd-ns flux
Then check the existence of the lasample namespace:

$ kubectl get namespaces
Then check that the Nginx deployment is running:

$ kubectl get pods --namespace=lasample
We should see the deployment running, with two replicas.

Conclusion
We're finished. We've installed Flux and configured it to work with a Kubernetes cluster and a GitHub repository. Congratulations!
