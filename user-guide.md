# Helm to Operator Codegen Sdk
The "Helm to Operator Codegen SDK" offers a streamlined solution for translating existing Helm charts into Kubernetes operators with minimal effort and cost.

By utilizing the Helm to Operator Codegen SDK, users can efficiently convert existing Helm charts into Kubernetes operators. This transition enables users to leverage the advanced capabilities provided by Kubernetes operators, such as enhanced automation, lifecycle management, and custom logic handling. Overall, the SDK facilitates a smooth migration process, empowering users to embrace the operator model for managing their Kubernetes resources effectively.

## Excercise: Deploying Free5gc using operator
In the following exercise, the objective is to convert the Free5gc Helm chart to Go code suitable for a Kubernetes operator using the sdk. Once the conversion is complete, the generated Go code will be used to deploy all the resources defined in the Free5gc Helm chart using a Kubernetes operator.

### Step 0: Prerequisite
1. GoLang Version: 1.21
2. Helm : v3.9.3
3. Kubebuilder
4. A Kubernetes Cluster with Calico CNI and multus-cni plugin (Can Refer [here](https://medium.com/rahasak/deploying-5g-core-network-with-free5gc-kubernets-and-helm-charts-29741cea3922), Before "Deploy Helm-Chart Part" )
5. Go Packages:
```
# Clone the Repo
cd nephio-sdk/helm-to-operator-codegen-sdk/
go mod tidy
```

### Step 1: Convert the helm-chart to Go-Code using Helm-to-operator-codegen-sdk
Currently, only Local-Helm charts are supported by sdk, Therefore, the first step would be to download the free5gc-Helm-Chart. (Refer [here](https://github.com/Orange-OpenSource/towards5gs-helm/tree/main))

To initiate the conversion process using the SDK, you can use the following command:
```
go run main.go <path_to_local_helm_chart> <namespace> <logging-level>
where:
    <path_to_local_helm_chart>: Path to your local chart, the folder must contain a chart.yaml file.
    <namespace>: The namespace you want to deploy the resources
    <logging-level>: debug, info (default), error, warn

```


#### Example Run 
```
go run main.go /home/ubuntu/free5gccharts/towards5gs-helm/charts/free5gc/charts/free5gc-amf/ free5gcns info
```
<details>
<summary>The output is similar to:</summary>

```console
INFO[0000]  ----------------- Converting Helm to Yaml --------------------------
WARN[0000] Duplication Detected in Struct Mapping | For Preconditions
WARN[0000] Duplication Detected in Struct Mapping | For ConditionStatus
WARN[0000] Duplication Detected in Enum Mapping | For ConditionStatus
INFO[0000] CurFile --> | temp/templated/free5gc-amf/templates/amf-configmap.yaml
INFO[0000]  Current KRM Resource| Kind : ConfigMap| YamlFilePath : temp/templated/free5gc-amf/templates/amf-configmap.yaml
INFO[0000]       Converting Runtime to Json Completed
INFO[0000]       Converting Json to String Completed
INFO[0000] CurFile --> | temp/templated/free5gc-amf/templates/amf-deployment.yaml
INFO[0000]  Current KRM Resource| Kind : Deployment| YamlFilePath : temp/templated/free5gc-amf/templates/amf-deployment.yaml
INFO[0000]       Converting Runtime to Json Completed
INFO[0000]       Converting Json to String Completed
INFO[0000] CurFile --> | temp/templated/free5gc-amf/templates/amf-hpa.yaml
ERRO[0000] Unable to convert yaml to unstructured |Object 'Kind' is missing in 'null'
INFO[0000] CurFile --> | temp/templated/free5gc-amf/templates/amf-ingress.yaml
ERRO[0000] Unable to convert yaml to unstructured |Object 'Kind' is missing in 'null'
INFO[0000] CurFile --> | temp/templated/free5gc-amf/templates/amf-n2-nad.yaml
INFO[0000] Kind | NetworkAttachmentDefinition Would Be Treated as Third Party Kind
INFO[0000]       Converting Unstructured to String Completed
INFO[0000] CurFile --> | temp/templated/free5gc-amf/templates/amf-service.yaml
INFO[0000]  Current KRM Resource| Kind : Service| YamlFilePath : temp/templated/free5gc-amf/templates/amf-service.yaml
INFO[0000]       Converting Runtime to Json Completed
INFO[0000]       Converting Json to String Completed
INFO[0000] ----------------- Writing GO Code ---------------------------------
INFO[0000] ----------------- Program Run Successful| Summary ---------------------------------
INFO[0000] Deployment            |1
INFO[0000] NetworkAttachmentDefinition           |1
INFO[0000] Service               |1
INFO[0000] ConfigMap             |1
```
</details>


The generated Go-Code would be written to the "outputs/generated_code.go" file

The Generated Go-Code shall contain the following pluggable functions:

A) Pluggable functions
1. CreateAll():  When called, it will create all the k8s resources(services, deployment) on the kubernetes cluster. (Note: For your Reconciler to call the function, Replace `YourKindReconciler` with the type of your Reconciler)
2. DeleteAll(): When called, it will delete all the k8s resources(services, deployment) on the kubernetes cluster. (Note: For your Reconciler to call the function, Replace `YourKindReconciler` with the type of your Reconciler)
3. Getxxx(): Shall return the list of a particular resource.
    1. GetService(): Shall return the list of all services.
    2. GetDeployment(): Shall return the list of all deployments. & so on

B) Helper Functions: (For internal use only)
1. deleteMeAfterDeletingUnusedImportedModules: This function is included in the generated Go code to handle the scenario where a module is imported but not used. Once the user has removed the non-required modules from the import statements, they can safely delete this function as well.
2. Pointer Fxns: `intPtr(), int16Ptr(), int32Ptr(), int64Ptr(), boolPtr(), stringPtr()`: Takes the value and returns the pointer to that value.
3. getDataForSecret: This function takes the encodedVal of Secret, decodes it, and returns



### Step 2: Using kubebuilder to develop the operator
Please refer [here](https://book.kubebuilder.io/quick-start) to develop & deploy the operator.

After the basic structure of the operator is created, users can proceed to add their business logic
The `CreateAll()` and `DeleteAll()` functions generated by the SDK can be leveraged for Day-0 resource deployments, allowing users to easily manage the creation and deletion of resources defined in the Helm chart. By integrating their business logic with these functions, users can ensure that their operator effectively handles resource lifecycle management and orchestration within a Kubernetes environment.

In the end, all the resources created could be checked by:
`kubectl get pods -n free5gcns`

<details>
<summary>The output Should be: </summary>

```


```
</details>

