<p align="center">
<a href="https://github.com/ankitcharolia/visitors-operator/actions/workflows/ci.yml"><img src="https://github.com/ankitcharolia/visitors-operator/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
<a href="https://github.com/ankitcharolia/visitors-operator/actions/workflows/release.yml"><img src="https://github.com/ankitcharolia/visitors-operator/actions/workflows/release.yml/badge.svg" alt="Release"></a>
<a href="https://github.com/ankitcharolia/visitors-operator/actions/workflows/helm.yml"><img src="https://github.com/ankitcharolia/visitors-operator/actions/workflows/helm.yml/badge.svg" alt="Helm"></a>
<a href="https://github.com/ankitcharolia/visitors-operator/actions/workflows/manifests.yml"><img src="https://github.com/ankitcharolia/visitors-operator/actions/workflows/manifests.yml/badge.svg" alt="Manifests"></a>
<a href="https://github.com/ankitcharolia/visitors-operator/actions/workflows/olm-helm.yml"><img src="https://github.com/ankitcharolia/visitors-operator/actions/workflows/olm-helm.yml/badge.svg" alt="OLM Helm"></a>
</p>

<p align="center">
<a href="https://goreportcard.com/report/github.com/ankitcharolia/visitors-operator"><img src="https://goreportcard.com/badge/github.com/ankitcharolia/visitors-operator" alt="Go Report Card"></a>
<a href="https://pkg.go.dev/github.com/ankitcharolia/visitors-operator"><img src="https://pkg.go.dev/badge/github.com/ankitcharolia/visitors-operator.svg" alt="Go Reference"></a>
</p>

# Visitors Operator
we implement an Operator in Go which will define some Custom Resource to control and deploy a 3-tier app:

* Frontend: React app from docker.io/jdob/visitors-webui:1.0.0

* Backend: Python app from docker.io/jdob/visitors-service:1.0.0

* DB: MySQL 5.7 from docker.io/library/mysql:5.7

### Scaffold the new operator
```bash
operator-sdk init --domain redhat.com --repo github.com/ankitcharolia/visitors-operator
```

### Generate an API
```shell
operator-sdk create api --group=app --version=v1 --kind=VisitorsApp --resource --controller
```

###  Invoke the controller-gen utility to update the api/v1/zz_generated.deepcopy.go file to ensure our API’s Go type definitons implement the runtime.Object interface that all Kind types must implement.
```shell
make generate
```

### Generate CRDs
```shell
make manifests
```

### Run your operator locally
```shell
make install run
```
```bash
acharolia@ankitcharolia:~/review/visitors-operator$ make run install 
test -s /home/acharolia/review/visitors-operator/bin/controller-gen && /home/acharolia/review/visitors-operator/bin/controller-gen --version | grep -q v0.12.0 || \
GOBIN=/home/acharolia/review/visitors-operator/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.12.0
/home/acharolia/review/visitors-operator/bin/controller-gen rbac:roleName=manager-role crd webhook paths="./..." output:crd:artifacts:config=config/crd/bases
/home/acharolia/review/visitors-operator/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
go fmt ./...
go vet ./...
go run ./cmd/main.go
2024-03-01T19:38:57+01:00	INFO	controller-runtime.metrics	Metrics server is starting to listen	{"addr": ":8080"}
2024-03-01T19:38:57+01:00	INFO	setup	starting manager
2024-03-01T19:38:57+01:00	INFO	starting server	{"path": "/metrics", "kind": "metrics", "addr": "0.0.0.0:8080"}
2024-03-01T19:38:57+01:00	INFO	Starting server	{"kind": "health probe", "addr": "0.0.0.0:8081"}
2024-03-01T19:38:57+01:00	INFO	Starting EventSource	{"controller": "visitorsapp", "controllerGroup": "app.redhat.com", "controllerKind": "VisitorsApp", "source": "kind source: *v1.VisitorsApp"}
2024-03-01T19:38:57+01:00	INFO	Starting Controller	{"controller": "visitorsapp", "controllerGroup": "app.redhat.com", "controllerKind": "VisitorsApp"}
2024-03-01T19:38:57+01:00	INFO	Starting workers	{"controller": "visitorsapp", "controllerGroup": "app.redhat.com", "controllerKind": "VisitorsApp", "worker count": 1}
2024-03-01T19:39:36+01:00	INFO	Reconciling VisitorsApp	{"controller": "visitorsapp", "controllerGroup": "app.redhat.com", "controllerKind": "VisitorsApp", "VisitorsApp": {"name":"visitorsapp-sample","namespace":"default"}, "namespace": "default", "name": "visitorsapp-sample", "reconcileID": "8f0f79ce-254d-434d-b3b5-19788b1d25d7", "Request.Namespace": "default", "Request.Name": "visitorsapp-sample"}
2024-03-01T19:39:36+01:00	INFO	visitor-operator	Creating a new secret	{"Secret.Namespace": "default", "Secret.Name": "mysql-auth"}
2024-03-01T19:39:36+01:00	INFO	visitor-operator	Creating a new Deployment	{"Deployment.Namespace": "default", "Deployment.Name": "mysql"}
2024-03-01T19:39:36+01:00	INFO	visitor-operator	Creating a new Service	{"Service.Namespace": "default", "Service.Name": "mysql-service"}
2024-03-01T19:39:36+01:00	INFO	MySQL isn't running, waiting for 5s	{"controller": "visitorsapp", "controllerGroup": "app.redhat.com", "controllerKind": "VisitorsApp", "VisitorsApp": {"name":"visitorsapp-sample","namespace":"default"}, "namespace": "default", "name": "visitorsapp-sample", "reconcileID": "8f0f79ce-254d-434d-b3b5-19788b1d25d7"}
```

### Apply a Custom Resource
```shell
kubectl apply -f config/samples/app_v1_visitorsapp.yaml
```
```shell
acharolia@ankitcharolia:~/review/visitors-operakubectl get svc
NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes                            ClusterIP   10.96.0.1        <none>        443/TCP          6h58m
mysql-service                         ClusterIP   None             <none>        3306/TCP         6m34s
visitorsapp-sample-backend-service    NodePort    10.98.74.235     <none>        8000:30685/TCP   6m4s
visitorsapp-sample-frontend-service   NodePort    10.105.255.210   <none>        3000:30686/TCP   6m4s

acharolia@ankitcharolia:~/review/visitors-operator$ kubectl get po
NAME                                           READY   STATUS    RESTARTS   AGE
mysql-64b7bdd7d9-jdpg7                         1/1     Running   0          6m38s
visitorsapp-sample-backend-5c6f9b56cc-6tp9b    1/1     Running   0          6m8s
visitorsapp-sample-frontend-6db6bf5df8-2848r   1/1     Running   0          6m8s

acharolia@ankitcharolia:~/review/visitors-operator$ kubectl get visitorsapps
NAME                 AGE
visitorsapp-sample   15m
```

### On Minikube, get Minikube IP and access the app
```bash
IP=$(minikube ip -p operators)
PORT=$(kubectl get service/visitorsapp-sample-frontend-service -o jsonpath="{.spec.ports[*].nodePort}")
curl $IP:$PORT
```

### Build and Push the Operator
```shell

```

**NOTE:** This will invoke controller-gen utility to generate the CRD manifests at config/crd/bases/app.redhat.com_visitorsapps.yaml.

### Controllers
Controllers are core components in Kubernetes and is where your operator logic takes place.

The reconcile function is responsible for enforcing the desired CR state on the actual state of the system. It runs each time an event occurs on a watched CR or resource, and will return some value depending on whether those states match or not.

In this way, every Controller has a Reconciler object with a Reconcile() method that implements the reconcile loop.


# References
* [Redhat Go Operator Tutorial](https://redhat-scholars.github.io/operators-sdk-tutorial/template-tutorial/04-go.html#init)
* [Operator SDK Quickstart](https://sdk.operatorframework.io/docs/building-operators/golang/quickstart/)
