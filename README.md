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
The SDK currently employs the "runtime-object method" to handle Kubernetes resources whose structure is recognized by Kubernetes by default. Examples of such resources include Deployment, Service, and ConfigMap. Conversely, resources that are not inherently known to Kubernetes and require explicit installation or definition, such as Third-Party Custom Resource Definitions (CRDs) like NetworkAttachmentDefinition or PrometheusRule, are processed using the "unstructured-object" method. Such examples are given below:
```
// Runtime-Object Example
service1 := &corev1.Service{
		TypeMeta: metav1.TypeMeta{
			APIVersion: "v1",
			Kind:       "Service",
		},
		ObjectMeta: metav1.ObjectMeta{
			Name: "my-service",
		},
		Spec: corev1.ServiceSpec{
			Selector: map[string]string{
				"app.kubernetes.io/name": "MyApp",
			},
			Ports: []corev1.ServicePort{
				corev1.ServicePort{
					Port:     80,
					Protocol: corev1.Protocol("TCP"),
					TargetPort: intstr.IntOrString{
						IntVal: 9376,
					},
				},
			},
		},
	}

// Unstruct-Object Example
networkAttachmentDefinition1 := &unstructured.Unstructured{
		Object: map[string]interface{}{
			"apiVersion": "k8s.cni.cncf.io/v1",
			"kind":       "NetworkAttachmentDefinition",
			"metadata": map[string]interface{}{
				"name": "macvlan-conf",
			},
			"spec": map[string]interface{}{
				"config": "some-config",
			},
		},
	}
```

### Flow-3.1: KRM to Runtime-Object
The conversion process relies on the "k8s.io/apimachinery/pkg/runtime" package. Currently, only the API version "v1" is supported. The supported kinds for the Runtime Object method include:
`Deployment, Service, Secret, Role, RoleBinding, ClusterRoleBinding, PersistentVolumeClaim, StatefulSet, ServiceAccount, ClusterRole, PriorityClass, ConfigMap`

### Flow-3.2: Runtime-Object to JSON
Firstly, the SDK performs a typecast of the runtime object to its actual data type. For instance, if the Kubernetes Kind is "Service," the SDK typecasts the runtime object to the specific data type corev1.Service. Then, it conducts a Depth-First Search (DFS) traversal over the corev1.Service object using reflection. During this traversal, the SDK generates a JSON structure that encapsulates information about the struct hierarchy, including corresponding data types and values. This transformation results in a JSON representation of the corev1.Service object's structure and content.

```
// For A KRM Resource
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80

// The Converted JSON Representation looks-like the following: 
{
    "ObjectMeta": {
        "type": "v1.ObjectMeta",
        "val": {
            "Name": {
                "type": "string",
                "val": "my-service"
            }
        }
    },
    "Spec": {
        "type": "v1.ServiceSpec",
        "val": {
            "Ports": {
                "type": "[]v1.ServicePort",
                "val": [
                    {
                        "Port": {
                            "type": "int32",
                            "val": "80"
                        },
                        "Protocol": {
                            "type": "v1.Protocol",
                            "val": "TCP"
                        }
                    }
                ]
            }
        }
    }
}
// It shows the hierarchical structure along with the specific data types and corresponding values for each attribute
```




