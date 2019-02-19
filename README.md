# Kubernetes-dep

For following this repository, you need to install:

- minikube ([official guide](https://kubernetes.io/docs/tasks/tools/install-minikube/)):
  <br>`$ minikube start --vm-driver hyperv --hyperv-virtual-switch "virtual-switch-name"`  (using Hyper-V)
  <br \>`$ minikube start`  (using VirtualBox)

- helm ([official guide](https://helm.sh/)):
 <br \> `$ choco install kubernetes-helm` (using Chocolatey via Windows)

- git ([official guide](https://git-scm.com/download/win))

This repository is created for testing the ability of Kubernetes of autoscaling when an overloading occour. Infact, the Horizontal Pod Autoscaler (HPA) automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization.
The HPA is implemented as a Kubernetes API resource and a controller. The resource determines the behavior of the controller. The controller periodically adjusts the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by user.

Note: HPA does not apply to objects that can’t be scaled; e.g. DaemonSets.

From the most basic perspective, the Horizontal Pod Autoscaler controller operates on the ratio between desired metric value and current metric value (function Math.ceil()):

	desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]


Now, running PowerShell like admin, and paste the following commands:

  `$ git clone https://github.com/ariamore/Kubernetes-dep.git`
  <br \> for cloning this repo locally, then change directory to -->  `$ cd .\Kubernetes-dep\`

You need to link your scripting to the helm repo ([available here](https://github.com/bitnami/charts)).
Then, you need to paste this instruction -->
  `$ helm repo update`

Check if the metrics-server is enable or not --> `$ minikube addons list`
<br \> instead, enable metrics-server running -->  `$ minikube addons enable metrics-server`

Install your application using helm --><br \>
`$ helm install --name test -f values.yaml bitnami/mean`

Expose your application on internet using --><br \>
`$ minikube service test-mean`

You can see your application on minikube dashboard running --><br \> `$ minikube dashboard`

For allowing the application autoscale, run the following command:<br \>
`$ kubectl autoscale deployment test --cpu-percent=[max CPU percentage before scaling; e.g.=20] --max=[pod replica number; e.g.=5]`

For verifying the status of scaling of the application, run --><br \> `$ kubectl get hpa`
<br \>(you should see something like this)<br \>

	NAME         REFERENCE                     TARGET     MINPODS   MAXPODS   REPLICAS   AGE
	test-mean    Deployment/test-mean/scale    0% / 25%     1         5         1          30sec


Now, if you want to test what happen when load increased, send an infinite loop of queries to your service (run it in a different terminal):<br \>
	`$ kubectl run -i --tty load-generator --image=busybox /bin/sh`<br \>
  `$ while true; do wget -q -O- http://[your-local-environment]; done`

After this, you CPU usage level should be higher (then the fixed limit);<br \>
so your application should scaling up (running again `$ kubectl get hpa` ):

	NAME         REFERENCE                     TARGET       MINPODS   MAXPODS   REPLICAS   AGE
	test-mean    Deployment/test-mean/scale    75% / 25%    1         10        1          5m

For seeing your replicas, run -->
`$ kubectl get deployment test-mean`

	NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
	test-mean   5         5         5            5           2h

For ending the process, press <Ctrl + c> on PowerShell (where you send queries). After few minutes, CPU utilization dropped to 0, and so HPA autoscaled the number of replicas back down to 1.
