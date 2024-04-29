# Helm - Notes

There are major changes from Helm 2 to Helm 3. 
We use Helm v3.X in our practice (it is considered as latest stable at present)

1. Creating a simple service and deployment - "Hello World"
charts are just a bunch of text files. You can install and uninstall a chart.
By readinf the content the helm knows what to implement. Thats how we have to write them.
It should be a form of template, later we pass values to replace them.
We can create a template of all our definition ex: pod, deployment or service.

apart from the yaml file, we also have the chart.yaml that has info on the chart.
It is like metadata.
```yaml
#Service.yaml before helm templating
---
apiVersion: v1
kind: service
metadata:
  name: svc-hello-world
  labels:
    app: hello-world
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: hello-world
```

This should be made as template

```yaml

#Service.yaml after helm templating
---
apiVersion: v1
kind: Service
metadata:
  name: {{ include "hello-world.fullname" . }}
  labels:
    {{- include "hello-world.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "hello-world.selectorLabels" . | nindent 4 }}

```
Create a directory called templates, it should contain all the yaml definitions with place holders.

It must be a replacable and reusable piece of block in every definition to form a template. We have to keep that in mind while writing the definition.

Below is the directory structure for helm chart.
- templates (dir)     # All K8 manifest/resource definition files
    - pod.yaml
    - service.yaml
- values.yaml         # Configurable values
- Chart.yaml          # Chart information
- charts (dir)        # Dependency charts

## Chart.yaml
Helm 2 works with `apiVersion: v1`
Helm 3 works with `apiVersion: v2` so use this.

Example of chart.yaml
```yaml
apiVersion: v2
appVersion: 5.8.1
version: 12.1.25
name: wordpress
description: Web publishing platform for buildng blogs and website
type: application
dependencies:
  - condition: mariadb.enabled
    name: mariadb
    repository: https://charts.bitnami.com/bitnami
    version: 9.x.x
keywords:
  - application
  - blog
  - wordpress
maintainers:
  - email: container@bitnami.com
    name: Bitnami
home: https://github.com/bitnami/charts/.....
icon: <shttps://somefaviconpath>
```
Imoortant fields to consider.

- AppVersion: The version of the application thats inside of this chart. This example its wordpress.
- Version: Which is the version of the chart. 
- Name: Ofcourse, name of the chart
- Type: Two types of charts. 
   1. Application - is the default type
   2. Library - provides utilites
- Dependencies: If there are any dependencies to the application.
- Keywords: Just like labels.


# Helm Basics - commands

Running `helm --help` is enough to know commands.

- `helm search hub wordpress` - gives results from artifacthub.io which is public hub for helm
- `helm search wordpress` - search for chart in locally configured repos.

To add the repo you can use
```console
$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
                                               

$ helm search repo wordpress
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/wordpress       22.2.2          6.5.2           WordPress is the worlds most popular blogging ...
bitnami/wordpress-intel 2.1.31          6.1.1           DEPRECATED WordPress for Intel is the most popu...

````
Then we can install the wordpress with your app name `mywpapp`

```console
$ helm install mywpapp bitnami/wordpress
NAME: mywpapp
LAST DEPLOYED: Mon Apr 29 13:21:22 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: wordpress
CHART VERSION: 22.2.2
APP VERSION: 6.5.2

** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    mywpapp-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w mywpapp-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default mywpapp-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default mywpapp-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/


```

Now `helm list` gives us list of avaible charts

```console
$ helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                APP VERSION
mywpapp default         1               2024-04-29 13:21:22.2984709 +0530 IST   deployed        wordpress-22.2.2     6.5.2
```

Once you install the app. You can see releated workloads like pods, svc, deployments in your k8 namespace.

```console
$ kubectl get all
NAME                                   READY   STATUS     RESTARTS   AGE
pod/mywpapp-mariadb-0                  0/1     Init:0/1   0          4m1s
pod/mywpapp-wordpress-88d479bb-x4ktm   0/1     Init:0/1   0          4m1s

NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/kubernetes          ClusterIP      10.43.0.1       <none>        443/TCP                      2d17h
service/mywpapp-mariadb     ClusterIP      10.43.166.129   <none>        3306/TCP                     4m2s
service/mywpapp-wordpress   LoadBalancer   10.43.58.117    <pending>     80:31567/TCP,443:31973/TCP   4m2s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mywpapp-wordpress   0/1     1            0           4m2s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mywpapp-wordpress-88d479bb   1         1         0       4m1s

NAME                               READY   AGE
statefulset.apps/mywpapp-mariadb   0/1     4m2s


```

`helm repo update` does the update of repo from the source. It is suggested to update once in a while.

The biggest advantage of Helm is, when ever you want to remove this app from your namespace, 

you can simply clean up using a command 
`helm uninstall mywpapp` 


That removes entire app `mympapp` with its workloads.

## Helm customizing 

### chart parameters

Talking about my previous example appliction. Have you wonder, how the title of the application got created with name  `User's Blog!` ?

Its because the bitname/wordress chart was created such a way it takes variable of the name of blog and replaced it in the template.

```yaml
#Values.yaml default values
...
wordpressBlogName: User's Blog!
wordpressUserEmail: user@example.com
...
```

we can also override that using `--set` command while installing the chart. Like this

`helm install --set wordpressBlogName="Helm Sample Blog" wordpressUserEmail="john@test.com" mywpapp bitnami/wordpress`

This will create an instance with provided params.

what if you have more such custom params? 

You can use the `--values` option passing the file with custome values `custom-values.yaml` file in your local, like this

`helm install --values custom-values.yaml mywpapp bitnami/wordpress`

Make sure your custom-values file contains all the varaibles you want to override.


### Helm pull - chart files download

If you want to edit the values.yaml with out passing custom values. Also you want to custom the whole chart the way you want. We can use `pull` command.

`helm pull bitnami/wordpress`

That downloads archived (tar) copy of files. which you need to unartchive (untar) in to your local directory. Or directly you can call this command.

`helm pull --untar bitnami/wordpress`

Now if you do `ls wordpress` you see all files in the worpress chart in your local directory.

And then you can install it locally like this

`helm install my-custom-wpapp ./wordpress`

## Lifecycle Management

Each time we pull and install the repo creates a release. In the background it upgrade/downgrade/remove objects in the kubernetes layer.  
The life cycle of the Helm chart consists of 3 stages:
- Install
```console
$ helm repo add  bitnami https://charts.bitnami.com/bitnami
$ helm install my-web bitnami/nginx
```
- Upgrade
```console
$ helm history my-web
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Mon Apr 29 10:32:14 2024        superseded      nginx-12.0.4    1.22.0          Install complete
2               Mon Apr 29 10:32:18 2024        superseded      nginx-12.0.5    1.22.0          Upgrade complete
3               Mon Apr 29 10:32:23 2024        deployed        nginx-12.0.4    1.22.0          Upgrade complete

$ helm upgrade my-web bitnami/nginx --version 13
Release "dazzling-web" has been upgraded. Happy Helming!
NAME: dazzling-web
LAST DEPLOYED: Mon Apr 29 10:37:23 2024
NAMESPACE: default
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 13.2.34
APP VERSION: 1.23.4

$ helm history my-web
$REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Mon Apr 29 10:32:14 2024        superseded      nginx-12.0.4    1.22.0          Install complete
2               Mon Apr 29 10:32:18 2024        superseded      nginx-12.0.5    1.22.0          Upgrade complete
3               Mon Apr 29 10:32:23 2024        superseded      nginx-12.0.4    1.22.0          Upgrade complete
4               Mon Apr 29 10:37:23 2024        deployed        nginx-13.2.34   1.23.4          Upgrade complete
```
- Rollback
```console
$ helm rollback my-web
Rollback was a success! Happy Helming!

$ helm history my-web
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Mon Apr 29 10:32:14 2024        superseded      nginx-12.0.4    1.22.0          Install complete
2               Mon Apr 29 10:32:18 2024        superseded      nginx-12.0.5    1.22.0          Upgrade complete
3               Mon Apr 29 10:32:23 2024        superseded      nginx-12.0.4    1.22.0          Upgrade complete
4               Mon Apr 29 10:37:23 2024        superseded      nginx-13.2.34   1.23.4          Upgrade complete
5               Mon Apr 29 10:44:38 2024        deployed        nginx-12.0.4    1.22.0          Rollback to 3 
```
NOTE: We can rollback the app version or chart to previous version. But the data attached to volume like DB are not touched. So make sure you take timely backups of your volumes.

## Helm charts Anatomy
### Writing Helm charts

We can write any kubernetes application just like a installation wizard. 
Helm charts can do some extra things too. We have a way to restore. 

Create a chart directory by simply creating it commandline. It automatically create templates, chart, values and Readme files.

```console
$ helm create hello-world
Creating hello-world

# it creates a directory
$ cd hello-world
$ tree
.
+--- .helmignore
+--- Chart.yaml
+--- charts
+--- templates
|   +--- deployment.yaml
|   +--- hpa.yaml
|   +--- ingress.yaml
|   +--- NOTES.txt
|   +--- service.yaml
|   +--- serviceaccount.yaml
|   +--- tests
|   |   +--- test-connection.yaml
|   +--- _helpers.tpl
+--- values.yaml

```

But we need use all of them. So remove the unwanted and keep the important ones.

Variables that we pass from values.yaml will be replaced with `{{ varname }} ` in the template
For ex:

```sh
apiVersion: apps/v1
kind: Deployment
name: {{ .Release.Name }}-nginx
...
...
```
Here `{{ }}` is go template language

`.Release.Name` - first dot indicates the root level. and .Name indicates object key.

There are similar objects.

|  Release  |  Chart  |  Capabilities | Values |
|---|---|---|---|
| Release.Name  | Chart.Name  | Capabilities.KubeVersion | Values.replicaCount |
| Release.NameSpace | Chart.ApiVersion  | Capabilities.ApiVersions  | Values.image |
| Release.IsInstall | Chart.Version  | Capabilities.HelmVersion |  |
| Release.Revision  | Chart.Type  | Capabilities.GitCommit  |  |
| Release.Service  | Chart.Keyowords  | Capabilities.GoVersion  |  |
| Release.IsUpgrade | Chart.Home |  | |


There are serveral other variables available. Basically pass values from values.yml, capability from the host system.


