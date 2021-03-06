# Importing a managed cluster with the CLI

Under construction.

After you install Red Hat Advanced Cluster Management for Kubernetes, you are ready to import a cluster to manage.


  - [Prerequisites](#prerequisites)
  - [Supported architecture](#supported-architecture)
  - [Prepare for import](#prepare-for-import)
  - [Importing the multicluster-endpoint](#importing-the-multicluster-endpoint)
    
  **Note:** A hub cluster cannot manage another hub cluster.
    
## Prerequisites

* You must have an Red Hat Advanced Cluster Management for Kubernetes hub that is deployed and cluster that you want to manage.

* You need to install the Kubernetes CLI, `kubectl`. To install `kubectl`, see _Install and Set Up kubectl_ in the [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos).

  **Note:** Download the installation file for CLI tools from the console.

## Supported architecture

* Linux
* macOS

## Prepare for import

1. Log in to your _hub cluster_. Run the following command:
   
  ```
  oc login
  ```

2. Run the following command on the hub cluster to create the `oc create ns <cluster_namespace>` namespace:

  ```
  oc create -n ${CLUSTER_NAMESPACE} secret docker-registry quay-secret --docker-server=quay.io --docker-username=${DOCKER_USER} --docker-password=${DOCKER_PASS}
  ```
  
3. Edit the example ClusterRegistry cluster in the `/test/resources/test_cluster.yaml` file of the `open-cluster-management` [repository](https://github.com/open-cluster-management/rcm-controller/blob/master/test/resources/test_cluster.yaml)

4. Add the following `imagePullSecret`: `quay-secret in test_endpoint_config.yaml`

   - Create a ClusterRegistry cluster: `oc apply -f test_cluster.yaml`
   - Refer to the [cluster-registry](https://github.com/kubernetes/cluster-registry/blob/master/pkg/apis/clusterregistry/v1alpha1/types.go) for API definition
  
5. Create the endpoint configuration. Edit the [example EndpointConfig resource[/test/resources/test_endpoint_config.yaml](https://github.com/open-cluster-management/rcm-controller/blob/master/test/resources/test_endpoint_config.yaml). 
  
6. Refer to [/pkg/apis/multicloud/v1alpha1/endpointconfig_types.go](https://github.com/open-cluster-management/rcm-controller/blob/master/pkg/apis/multicloud/v1alpha1/endpointconfig_types.go) and [endpoint-operator - /pkg/apis/multicloud/v1beta1/endpoint_types.go](https://github.com/open-cluster-management/endpoint-operator/blob/master/pkg/apis/multicloud/v1beta1/endpoint_types.go) for API definition.
  
7. Create a Multicloud EndpointConfig. Run the following command: 

  ```
  oc apply -f test_endpoint_config.yaml
  ```

The ClusterController takes the following actions:

- EndpointConfig creation triggers `Reconcile()` in [/pkg/controllers/endpointconfig/endpointconfig_controller.go](https://github.com/open-cluster-management/rcm-controller/blob/master/pkg/controller/endpointconfig/endpointconfig_controller.go).
  
- Controller uses information in EndpointConfig to generate a secret named `{cluster-name}-import`.
  
- The `{cluster-name}-import` secret contains the `import.yaml` that the user applies to a managed cluster to install `multicluster-endpoint`.

## Importing the multicluster-endpoint

1. Obtain the `import.yaml` that was generated by the cluster controller.

- For macOS, run the following command:

  ```bash
  kubectl get secret ${cluster_name}-import -n ${cluster_namespace} -o jsonpath={.data.import\\.yaml} | base64 -D > import.yaml
  ```

- For Linux, run the following command:

  ```bash
  kubectl get secret ${cluster_name}-import -n ${cluster_namespace} -o jsonpath={.data.import\\.yaml} | base64 -d > import.yaml
  ```

2. Login to your target managed cluster.
  
3. Apply the `import.yaml` generated in previous step. Run the following command:
  
  ```
  kubectl apply -f import.yaml --validate=false
  ```

4. Validate the pod status on the target managed cluster. Run the following command:
   
  ```
  kubectl get pod -n multicluster-endpoint
  ```

5. Check the cluster on the hub cluster. Run the following command:
   
   ```
   kubectl get cluster --all-namespaces
   ```
