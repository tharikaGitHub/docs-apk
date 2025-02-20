## Deploy in OpenShift

APK supports OpenShift, which is a Kubernetes distribution with additional features and functionalities provided by Red Hat. The deployment process in OpenShift is similar to Kubernetes, with a few adjustments. Below are the steps to deploy APK in an OpenShift cluster.

### Prerequisites
* Access to an OpenShift cluster.
* Helm installed and initialized on your local machine. If you want to run in your local machine, you need to setup a RedHat CodeReady Container.
* OpenShift Client(oc) installed.
* Basic knowledge of using OpenShift and Helm.

### Deployment Steps
1. Create a new helm repository with the latest apk release using the following command. Let’s consider the ```<repository-name>``` as ```wso2apk``` for this guide.

	=== "Format"
		```
		helm repo add <repository-name> <link-to-latest-apk-release>
		```
	
	=== "Command"
		```
		helm repo add wso2apk https://github.com/wso2/apk/releases/download/0.0.1-m13
		```

2. Execute the following command to update the helm repositories.

      ```console
      helm repo update
      ```

3.  Create a namespace in OpenShift. Consider the ```<namespace>``` as ```apk``` for this guide.

	=== "Format"
		```
		oc new-project <namespace>
		```
	
	=== "Command"
		```
		oc new-project apk
		```

4. Install the APK components and start WSO2 API Platform For Kubernetes. Consider ```apk-test``` as the ```<chart-name>``` for this guide. As the ```--version``` of this command, use the version of the release you used in point 1 above. It will take a few minutes for the deployment to complete.

	=== "Format"
		```
		helm install <chart-name> <repository-name>/apk-helm --version <verison-of-APK> -n <namespace>
		```
	
	=== "Command"
		```
		helm install apk-test wso2apk/apk-helm --version 0.0.1-m13 -n apk
		```

5.  Now you can verify the deployment by executing the following command. You will see the status of the pods as follows once completed.

    === "Format"
        ```
        oc get pods -n <namespace>
        ```

    === "Command"
        ```
        oc get pods -n apk
        ```

6. To access the deployment through your local machine"

    1. Identify the `router-service` external IP address.
        ```console
        oc get svc -n apk | grep router-service
        ```
    2. Port forward router service to localhost.
        ```console
        oc port-forward svc/apk-test-wso2-apk-router-service -n apk 9095:9095
        ```

!!! info "(Note) Handle security context contraints"

	OpenShift applies Security Context Constraints (SCC) to restrict the actions that pods can perform. Part of this restriction involves removing fsGroup and runAsUser from the pod definition. However, there is a known issue with Helm, where setting null keys does not remove keys in nested subcharts. This can lead to failures in Redis and Postgres deployments. To work around this issue, you need to pass the null parameters as command-line arguments when installing APK using Helm. Following command shows how to do this.

		helm install apk-test wso2apk/apk-helm --version 0.0.1-m13 -n apk --set redis.master.podSecurityContext.fsGroup=null --set redis.master.containerSecurityContext.runAsUser=null --set postgresql.primary.podSecurityContext.fsGroup=null --set postgresql.primary.containerSecurityContext.runAsUser=null

