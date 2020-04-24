## Introduction to creating operators 
operator-sdk uses operator runtime (series of go packages, helps creation of operators easy)

### **Creating a New GO project operator**
1. <b>operator-sdk new</b> : Initializing a operator project using go
>  ```
> operator-sdk new demo --type=go
> 
> flags:
>     --type string       Type of operator to initialize (choices: "go", "ansible" or "helm") (default "go")
> ```
By default the operator is Namespace scoped, 👉 <a href="https://github.com/operator-framework/operator-sdk/blob/master/doc/operator-scope.md">cluster scoped operator</a>

### Adding a new custom API
1. <b>operator-sdk add api</b> : Adds a new api definition under pkg/apis, this links the code aware of the CRD that will be used with this api.
For Go-based operators:
    1. Creates the api definition for a new custom resource under pkg/apis.
    2. By default, this command runs Kubernetes deepcopy and CRD generators on tagged types in all paths under pkg/apis. Go code is generated under pkg/apis///zz_generated.deepcopy.go. Generation can be disabled with the --skip-generation flag for Go-based operators.

> ```
> operator-sdk add api --api-version=app.csye7374.com/v1alpha1 --kind=PodSet --skip-generation
> 
> flags:
>    --kind string       Kubernetes resource Kind name. (e.g AppService, PodSet, anything)
>    --api-version       Kubernetes APIVersion that has a format of $GROUP_NAME/$VERSION (e.g app.example.com/v1alpha1)
> ```

### Adding a new custom API
1. <b>operator-sdk generate</b> : command invokes a specific generator to generate code or manifests on disk.
    1. <b>operator-sdk generate k8s</b> : Generates Kubernetes code for custom resource 
    generates code for custom resources given the API specs in pkg/apis// directories to comply with kube-API requirements. Go code is generated under pkg/apis///zz_generated.deepcopy.go. Example:
    > ```
    > operator-sdk generate k8s [flags]
    > ```
    Example:
    >> ```
    >> $ tree pkg/apis
    >> pkg/apis/
    >> └── app
    >>     └── v1alpha1
    >>         ├── zz_generated.deepcopy.go
    >> ```

    2. <b>operator-sdk generate crds</b> : Generates CRDs for API's
    generates CRDs or updates them if they exist, under deploy/crds/<api-version>_<kinds>_crd.yaml; OpenAPI V3 validation YAML is generated as a 'validation' object
    > ```
    > operator-sdk generate crds [flags]
    > 
    > flags:
    > --crd-version string       CRD version to generate (default "v1beta1")
    > ```
    Example:
    >> ```
    >> $ tree deploy/crds
    >> ├── deploy/crds/app.csye7374.com_v1alpha1_podset_cr.yaml
    >> ├── deploy/crds/app.csye7374.com_podsets_crd.yaml
    >> ```

### Adding a new controller
1. <b>operator-sdk add controller</b> : Add a new controller package to your operator project.
This command creates a new controller package under pkg/controller/ that, by default, reconciles on a custom resource for the specified apiversion and kind. The controller will expect to use the custom resource type that should already be defined under pkg/apis// via the "operator-sdk add api" command.
> ```
> operator-sdk add controller --api-version=app.csye7374.com/v1alpha1 --kind=PodSet
> 
> flags:
>    --kind string       Kubernetes resource Kind name. (e.g AppService, PodSet, anything)
>    --api-version       Kubernetes APIVersion that has a format of $GROUP_NAME/$VERSION (e.g app.example.com/v1alpha1)
> ```
Example:
>> ```
>> $ tree pkg/controller
>> pkg/controller/
>> ├── add_appservice.go
>> ├── appservice
>> │   └── appservice_controller.go
>> └── controller.go
>> ```


### Running the operator
1. <b>operator-sdk run</b> : Run an Operator in a variety of environments
This command will run or deploy your Operator in two different modes: locally and using OLM. These modes are controlled by setting --local and --olm run mode flags. Each run mode has a separate set of flags that configure 'run' for that mode. Run 'operator-sdk run --help' for more information on these flags.
> ```
> operator-sdk run --local --namespace=demo
> 
> flags:
>    --local                        The operator will be run locally by building the operator binary with the ability to access a kubernetes cluster using a kubeconfig file. Cannot be set with another run-type flag.
>    --kubeconfig string            The file path to kubernetes configuration file. Defaults to location specified by $KUBECONFIG, or to default file rules if not set
>    --namespace string             (Deprecated: use --watch-namespace instead.)The namespace where the operator watches for changes.
>
>    --olm                          The operator to be run will be managed by OLM in a cluster. Cannot be set with another run-type flag
>    --olm-namespace string         [olm only] The namespace where OLM is installed (default "olm")
>    --operator-namespace string    [olm only] The namespace where operator resources are created. It must already exist in the cluster or be defined in a manifest passed to --include
> ```


## Information regarding Generating CRD (<a href="https://book-v1.book.kubebuilder.io/beyond_basics/generating_crd.html">Documentation</a>)
<b>operator-sdk generate crds</b> generates manifests for CustomResourceDefinitions. <b>operator-sdk generate crds</b> reads kubebuilder annotations of the form // +kubebuilder:something... defined as Go comments in the <your-api-kind>_types.go file under pkg/apis/... to produce the CRD manifests. The section below explains various supported annotations.

1. Validation 
One can specify validation for a field by annotating the field with kubebuilder annotation which is of the form // +kubebuilder:validation:<key=value>. Currently, supporting keys are <b>Maximum</b>, <b>Minimum</b>, <b>MaxLength</b>, <b>MinLength</b>, <b>MaxItems</b>, <b>MinItems</b>, <b>UniqueItems</b>, <b>Enum</b>, <b>Pattern</b>, <b>ExclusiveMaximum</b>, <b>ExclusiveMinimum</b>, <b>MultipleOf</b>, <b>Format</b>. 👉 <a href="https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#properties">list of keys</a>

Example:
>> ```
>> type ToySpec struct {
>> 
>>     // +kubebuilder:validation:Maximum=100
>>     // +kubebuilder:validation:Minimum=1
>>     // +kubebuilder:validation:ExclusiveMinimum=true
>>     Power  float32 `json:"power,omitempty"`
>> 
>>     Bricks int32   `json:"bricks,omitempty"`
>>     // +kubebuilder:validation:MaxLength=15
>>     // +kubebuilder:validation:MinLength=1
>>     // +kubebuilder:validation:Pattern=\w*(\s\w*)*\d*
>>     Name string `json:"name,omitempty"`
>> 
>>     // +kubebuilder:validation:MaxItems=500
>>     // +kubebuilder:validation:MinItems=1
>>     // +kubebuilder:validation:UniqueItems=false
>>     Knights []string `json:"knights,omitempty"`
>> 
>>     // +kubebuilder:validation:Enum=Lion,Wolf,Dragon
>>     Alias string `json:"alias,omitempty"`
>> 
>>     // +kubebuilder:validation:Enum=1,2,3
>>     Rank    int    `json:"rank"`
>> }

2. Additional printer columns
kubectl uses server-side printing. The server decides which columns are shown by the kubectl get command. You can customize these columns using a CustomResourceDefinition. To add an additional column, add a comment with the following annotation format just above the struct definition of the Kind.
Format: // +kubebuilder:printcolumn:name="Name",type="type",JSONPath="json-path",description="desc",priority="priority",format="format"
Note that description, priority and format are optional. Refer to the additonal printer columns docs to learn more about the values of name, type, JsonPath, description, priority and format. 👉 <a href="https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#data-types">list of types</a>

The following example adds the Spec, Replicas, and Age columns:
>> ```
>> // +kubebuilder:printcolumn:name="Spec",type="integer",JSONPath=".spec.cronSpec",description="status of the kind"
>> // +kubebuilder:printcolumn:name="Replicas",type="integer",JSONPath=".spec.Replicas"
>> // +kubebuilder:printcolumn:name="Age",type="date",JSONPath=".metadata.creationTimestamp"
>> type CronTab struct {
>>     metav1.TypeMeta   `json:",inline"`
>>     metav1.ObjectMeta `json:"metadata,omitempty"`
>> 
>>     Spec   CronTabSpec   `json:"spec,omitempty"`
>>     Status CronTabStatus `json:"status,omitempty"`
>> }

3. Subresource
    1. Status : To enable /status subresource, annotate the kind with // +kubebuilder:subresource:status format
    2. Scale : To enable /scale subresource, annotate the kind with // +kubebuilder:subresource:scale:specpath=<jsonpath>,statuspath=<jsonpath>,selectorpath=<jsonpath> format.
    Scale subresource annotation contains three fields: specpath, statuspath and selectorpath:
        1. <b>specpath</b> refers to specReplicasPath attribute of Scale object, and value jsonpath defines the JSONPath inside of a custom resource that corresponds to Scale.Spec.Replicas. This is a required field.
        2. <b>statuspath</b> refers to statusReplicasPath attribute of Scale object. and the jsonpath value of it defines the JSONPath inside of a custom resource that corresponds to Scale.Status.Replicas. This is a required field.
        3. <b>selectorpath</b> refers to labelSelectorPath attribute of Scale object, and the value jsonpath defines the JSONPath inside of a custom resource that corresponds to Scale.Status.Selector. This is an optional field.
    >> ```
    >> // +kubebuilder:subresource:scale:specpath=.spec.replicas,statuspath=.status.replicas
    >> ```



## Concepts for Controller
### Owners and dependents
1. Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. The owned objects are called dependents of the owner object. Every dependent object has a metadata.ownerReferences field that points to the owning object.
2. Sometimes, Kubernetes sets the value of ownerReference automatically. For example, when you create a ReplicaSet, Kubernetes automatically sets the ownerReference field of each Pod in the ReplicaSet. In 1.8, Kubernetes automatically sets the value of ownerReference for objects created or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job and CronJob.
Example:
>> ```
>> apiVersion: v1
>> kind: Pod
>> metadata:
>>   ...
>>   ownerReferences:
>>   - apiVersion: apps/v1
>>     controller: true
>>     blockOwnerDeletion: true
>>     kind: ReplicaSet
>>     name: my-repset
>>     uid: d9607e19-f88f-11e6-a518-42010a800195
>>   ...
>> ```
