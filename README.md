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
