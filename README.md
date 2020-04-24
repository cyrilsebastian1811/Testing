## Introduction to creating operators 
operator-sdk uses operator runtime (series of go packages, helps creation of operators easy)

### **Creating a New GO project operator**
**1. operator-sdk new:** Initializing a operator project using go
>  ```
> operator-sdk new demo --type=go
> 
> flags:
>     --type string       Type of operator to initialize (choices: "go", "ansible" or "helm") (default "go")
> ```
By default the operator is Namespace scoped, 👉 <a href="https://github.com/operator-framework/operator-sdk/blob/master/doc/operator-scope.md">cluster scoped operator</a>

### Adding a new custom API
**1. operator-sdk add api:** Adds a new api definition under pkg/apis, in order for the code to be aware of the CRD that will be used with this api.\
For Go-based operators:
    **1.** Creates the api definition for a new custom resource under pkg/apis.
    **2.** By default, this command runs Kubernetes deepcopy and CRD generators on tagged types in all paths under pkg/apis. Go code is generated under pkg/apis///zz_generated.deepcopy.go. Generation can be disabled with the --skip-generation flag for Go-based operators.

> ```
> operator-sdk add api --api-version=app.csye7374.com/v1alpha1 --kind=PodSet --skip-generation
> 
> flags:
>    --kind string       Kubernetes resource Kind name. (e.g AppService, PodSet, anything)
>    --api-version       Kubernetes APIVersion that has a format of $GROUP_NAME/$VERSION (e.g app.example.com/v1alpha1)
> ```


### Generators
**1. operator-sdk generate:** command invokes a specific generator to generate code or manifests on disk.
    **1. operator-sdk generate k8s:** Generates Kubernetes code for custom resource given the API specs in pkg/apis/ directories to comply with kube-API requirements. Go code is generated under pkg/apis///zz_generated.deepcopy.go
   > ```
   > operator-sdk generate k8s [flags]
   > ```
   **Example:**
   >> ```
   >> $ tree pkg/apis
   >> pkg/apis/
   >> └── app
   >>     └── v1alpha1
   >>         ├── zz_generated.deepcopy.go
   >> ```

   **2. operator-sdk generate crds:** Generates CRDs for API's or updates them if they exist, under deploy/crds/<api-version>_<kinds>_crd.yaml; OpenAPI V3 validation YAML is generated as a 'validation' object
   > ```
   > operator-sdk generate crds [flags]
   > 
   > flags:
   > --crd-version string       CRD version to generate (default "v1beta1")
   > ```
   **Example:**
   >> ```
   >> $ tree deploy/crds
   >> ├── deploy/crds/app.csye7374.com_v1alpha1_podset_cr.yaml
   >> ├── deploy/crds/app.csye7374.com_podsets_crd.yaml
   >> ```


### Adding a new controller
**1. operator-sdk add controller:** Add a new controller package to your operator project\
This command creates a new controller package under pkg/controller/ that, by default, reconciles on a custom resource for the specified apiversion and kind. The controller will expect to use the custom resource type that should already be defined under pkg/apis// via the "operator-sdk add api" command.
> ```
> operator-sdk add controller --api-version=app.csye7374.com/v1alpha1 --kind=PodSet
> 
> flags:
>    --kind string       Kubernetes resource Kind name. (e.g AppService, PodSet, anything)
>    --api-version       Kubernetes APIVersion that has a format of $GROUP_NAME/$VERSION (e.g app.example.com/v1alpha1)
> ```
**Example:**
>> ```
>> $ tree pkg/controller
>> pkg/controller/
>> ├── add_appservice.go
>> ├── appservice
>> │   └── appservice_controller.go
>> └── controller.go
>> ```


### Running the operator
**1. operator-sdk run:** Run an Operator in a variety of environments\
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
