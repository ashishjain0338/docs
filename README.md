# Helm to Operator Codegen Sdk
The "Helm to Operator Codegen SDK" offers a streamlined solution for translating existing Helm charts into Kubernetes operators with minimal effort and cost.

# The Flow Diagram
In a nutshell, Firstly, the Helm-Charts are converted to Yamls using the values provided in "values.yaml". Then, each Kubernetes Resource Manifest (KRM) in the YAML is translated into Go code, employing one of two methods.
1) If the resource is Runtime-Supported, it undergoes a conversion process where the KRM resource is first transformed into a Runtime Object, then into JSON, and finally into Go code.
2) Otherwise, if the resource is not Runtime-Supported, it is converted into an Unstructured Object and then into Go code.

After the conversion process, all the generated Go code is gathered and compiled into a single Go file. This resulting file contains functions that can be readily utilized by Kubernetes operators.

![alt Flow Diagram](https://github.com/ashishjain0338/docs/blob/main/Helm-To-Operator-Codegen-SDK.jpg)

-----
### Flow-1: Helm to Yaml
Helm to Yaml conversion is achieved by running the command
`helm template <chart>  --namespace <namespace>  --output-dir “temp/templated/”` Internally. As of now, It retrieves the values from default "values.yaml"

### Flow-2: Yaml Split
The SDK iterates over each YAML file in the "converted-yamls" directory. If a YAML file contains multiple Kubernetes Resource Manifests (KRM), separated by "---", the SDK splits the YAML file accordingly to isolate each individual KRM resource. This ensures that each KRM resource is processed independently.

### Runtime-Object and Unstruct-Object
The SDK currently employs the "runtime-object method" to handle Kubernetes resources whose structure is recognized by Kubernetes by default. Examples of such resources include Deployment, Service, and ConfigMap. Conversely, resources that are not inherently known to Kubernetes and require explicit installation or definition, such as Third-Party Custom Resource Definitions (CRDs) like NetworkAttachmentDefinition or PrometheusRule, are processed using the "unstructured-object" method.





