# Helm - Notes

Read in better view here : https://sudhakarm.github.io/helm_practice/


There are major changes from Helm 2 to Helm 3. 
We use Helm v3.X in our practice (it is considered as latest stable at present)

<p>Creating a simple service and deployment - "Hello World"
charts are just a bunch of text files. You can install and uninstall a chart.</p>
By reading the content the helm knows what to implement. Thats how we have to write them.
<p>
It should be a form of template, later we pass values to replace them.

We can create a template of all our definition ex: pod, deployment or service.
</p>


# Helm templates
apart from the yaml file, we also have the chart.yaml that has info on the chart.
It is like metadata. 

Note: #starting raw for gh-pages formatting curly braces. You have to put end raw in the end of the page.
{% raw %}

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


## Helm Basics - commands

Running `helm --help` is enough to know commands.
```console
$ helm --help
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts
...
Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  push        push a chart to remote
  registry    login to or logout from a registry
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information
```

### helm search
- `helm search hub wordpress` - gives results from artifacthub.io which is public hub for helm
- `helm search wordpress` - search for chart in locally configured repos.

### helm repo add
To add the repo you can use
```console
$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
                                               

$ helm search repo wordpress
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/wordpress       22.2.2          6.5.2           WordPress is the worlds most popular blogging ...
bitnami/wordpress-intel 2.1.31          6.1.1           DEPRECATED WordPress for Intel is the most popu...

````

### helm install
-  `helm install <yourReleaseName> bitnami/wordpress` will install wordpress from the repo you mention
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
### helm list
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

### helm update
`helm repo update` does the update of repo from the source. It is suggested to update once in a while.

The biggest advantage of Helm is, when ever you want to remove this app from your namespace, 

### helm uninstall
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
### --set
we can also override that using `--set` command while installing the chart. Like this

`helm install --set wordpressBlogName="Helm Sample Blog" wordpressUserEmail="john@test.com" mywpapp bitnami/wordpress`

This will create an instance with provided params.

what if you have more such custom params? 
### --values 
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


# Lifecycle Management

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

# Helm charts Anatomy
## Writing Helm charts

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


### Helm Lint 
When dealing with various files in the chart. There could be a chance of committing mistakes with whitspaces or wrong indentationa. In this case you have to use `helm lint <chartname>`

```console
$ helm lint ./hello-world/
==> Linting ./hello-world/
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

### Helm template validation (important)

If we are not sure we used wrong variable name or variable doesnt exist or how these values are translated. In this case we have to use `helm template <chartname>`. This will render us the clear output after merging values in to the template. Means the final yaml.

```console
$ helm template ./hello-world/
---
# Source: hello-world/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: release-name
  namespace: helloworld
  labels:
    app: release-name
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: release-name
---
# Source: hello-world/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-name
  namespace: helloworld
  labels:
    app: release-name
spec:
  replicas: 1
  selector:
    matchLabels:
      app: release-name
  template:
    metadata:
      labels:
        app: release-name
    spec:
      containers:
        - name: nginx
          image: "nginx:1.21.6-alpine"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP

```

### Helm Dryrun

Before installing the chart, if you want to make sure that all required fields are provided.
use `--dry-run` option to test that.

```console
$ helm install hello ./hello-world/ --dry-run
NAME: hello
LAST DEPLOYED: Tue Apr 30 17:28:46 2024
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: hello-world/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
  namespace: helloworld
  labels:
    app: hello
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: hello
---
# Source: hello-world/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  namespace: helloworld
  labels:
    app: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: nginx
          image: "nginx:1.21.6-alpine"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

This gives accurate chart definitions replacing release name as well.

### Helm Functions

In the the templates, we can write logical cases to take values from the variables using functions.
There are different types of functions. For example `upper` is a string function that converts the string to uppercase. If you want to conver the repository name of your deployment to be uppercase. You can simply use like this
```sh
#suppose value of image.repository is 'nginx'
image: {{ upper .Values.image.repository }}
#output 
image: NGINX


image: {{ quote .Values.image.repository }} 
#output 
image: "nginx"

image: {{ replace "x" "y" .Values.image.repository }} 
#output 
image: nginy
```
There are serveral other functions available in helm documentation
https://helm.sh/docs/chart_template_guide/function_list

Also, using `default` function we can pass the default value to take in case the variable is not in values.yml. Like below

`image: {{ default "nginx:1.16.0" .Values.image.repository}}`

The coalesce function takes a list of values and returns the first non-empty one.

```sh
Ex: 
coalesce 0 1 2

returns 1
```

### Helm pipelines

Just like pipes in linux, we can pass the output of one command/expression to the next stage using pipe `|`. In helm we call it as pipeline.

Chaining together multiple functions to express a series of transformations is known as a pipeline.

Taking the same example as above. We can send the value of image repository

```sh
# source: templates/deployment.yaml
image: {{ replace "ng" "bz" .Values.image.repository | upper }}
#output 
image: BZINX
```

### Helm Conditionals
In programming language we use `if` statement for conditional logic. Similarly we can set conditions in the helm templates. For example, our service yaml has the new label called `orgLabel` passed in the values.yaml.
write a condition only if that is present.

* Remember the condition block should have properindentation and also proper closure. if - else - end. 
* Space after the dash.
* dash indicates replace this block

```yaml
# eq is equals 
metadata: 
  name: {{ .Release.Name }}-nginx
  {{- if .Values.orgLabel }}
  labels:
    org: {{ .Values.orgLabel }}
  {{- end }}
  spec:
    ports:
      - port: 80
        name: http
  ...
```
### With blocks - Scopes

Suppose we have a configmap.yaml with data as dictionaty reading from values.yaml.
That king of hierarchy with scope. Here `dot` in the `.Values` indicates that is on the root level.

```yaml
# values.yaml
app:
  ui:
    bg: red
    fg: black
  db:
    name: "user"
    conn: "mongodb://localhost:27020/dbname"
```

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-appinfo`
data:
  background: {{ .Values.app.ui.bg }}
  foreground : {{ .Values.app.ui.fg }}
  database: {{ .Values.app.db.name }}
  connection: {{ .Values.app.db.conn }}
  chartname: {{ .Release.Name }}
```
This works by just regular template filling values. We can use `with` block for not repeating the references directly from the root level. Note that `$` also can be used, which indicates the root level even if you are inside with block.

The above configmap can be rewritten using with block like below

```yaml
# configmap.yaml with "with block"
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-appinfo
data:
  {{- with .Values.app }} # takes reference as .Values.app
    {{- with .ui }} # takes reference as .Values.app + .ui
  background: {{ .bg }}
  foreground : {{ .fg }}
    {{- end }} # end of ui
    {{- with .db }} # takes reference as .Values.app + .ui
  database: {{ .name }}
  connection: {{ .conn }} 
  chartname: {{ $Release.Name}} # even inside the db scope, it works with $ in prefix as root scope
    {{- end }} # end of .db
  {{- end }} # end of .Values.app
```

### Loops and Ranges
Just like in programming, we have loop for repeative function. Based on the number of iterations that function works.

Lets have a list of regions in teh values.yaml and our config map should add all feilds based on list/array depending on the size of array.

Ex: 

```yaml
# values.yaml
regions:
  - ohio
  - newyork
  - ontario
  - london
  - singapore
  - mumbai
```
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-appinfo
data:
  regions: 
  {{- range .Values.regions }} # loop starts depending on the range of items in regions list.
    - {{ . | quote }}
  {{- end }}  #end of loop
```

Another example
```yaml
# values.yaml
serviceAccount:
  # Specifies whether a service account should be created
  create: true
  annotations: {}
  name: webapp-sa
  labels: 
    tier: frontend
    type: web
    mode: proxy
```

```yaml
# serviceaccount.yaml
{{- with .Values.serviceAccount.create }}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ $.Values.serviceAccount.name }}
  labels:
    {{- range $key, $val := $.Values.serviceAccount.labels }}
    {{ $key }}: {{ $val }}
    {{- end }}
    app: webapp-color
{{- end }}
```

output of service account will be like

```yaml
---
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: webapp-sa
  labels:
    mode: proxy
    tier: frontend
    type: web
    app: webapp-color
```

## Named Templates

Some times we tend to use same set of lines repeated across various manifest files. For ex, the labels part in deployment and service files. 

```yml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }} 
  namespace: {{ .Values.namespace }}
  labels:
    app.kubernetes.io/name: {{ .Release.Name }} 
    app.kubernetes.io/instance: {{ .Release.Name }} 
...
```
We can put these repeating lines to a file called `_helper.tpl`. 
The underscore in the begining of the file name tells helm to avoid converting this files. 

We define these lines with a variable like this and can use it anywhere we like in template manifest types.

```yaml
# _helpers.tpl
{{- define "labels" }}
    app.kubernetes.io/name: {{ .Release.Name }}
    app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

```
usage in service file

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }} 
  namespace: {{ .Values.namespace }}
  labels:
    {{- template "labels" . }} # <-- dot gives current context
...
```
Since we are not giving the context to resolve .Release.Name in the _helpers.tpl file, we have to pass the context of root in this case like this `{{- template "labels" . }}`

Also, remember we have to think about indentation of the template as it should appear in the final manifest files. 

What happens if you use the same template in a deployment. Where you have labels in two levels.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Values.namespace }}
  labels: 
    app.kubernetes.io/name: {{ .Release.Name }}
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ .Release.Name }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ .Release.Name }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
  ...
```
It works as expected with correct indentation. But it fails to form the indent on the second label. So we use `indent 4` as a pipeline `|` to the template there to indent.



| Item | Type
| :------ | ------
| template | action 
| include |function 



Remember, a template is not a function but its an action. So it cannot take the pipe as that doesnt produce any output. So we have to use a function like this to `include` the labels. Show in the below example. `{{- include "labels" . | indent 4}}`

```yaml
# deployment.yaml after labels helper
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Values.namespace }}
  labels: 
    {{- template "labels" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "labels" . | indent 2}}
  template:
    metadata:
      labels:
        {{- include "labels" . | indent 4}}
    spec:
      containers:
  ...
```
## Chart Hooks
When ever we do `helm upgrade`, the process in the background takes a backup and then prepares for the version 2 changes. In this process it allows us to use hooks in various stages.
- Helm upgrade => Verify -> Render -> Upgrade

between these stages there are hooks
- Helm install => Verify -> Render -> [`pre-install Hook`] -> Install -> [`post-install Hook`]
- Helm delete => Verify -> Render -> [`pre-delete Hook`] -> Delete -> [`post-delete Hook`]
- Helm upgrade => Verify -> Render -> [`pre-upgrade Hook`] -> Upgrade -> [`post-upgrade Hook`]
- Helm rollback => Verify -> Render -> [`pre-rollback Hook`] -> rollback -> [`post-rollback Hook`]

### creating hooks
Lets have a script for `backup.sh` we want to run only want to run at once. One way to configure that as a startup pod.
But what if you want it to run pre-upgrade process?

you have to create an `annotation` to store data. 

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}
  annotations: # << Here we add the hooks>>
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: {{ .Release.Name }}
    spec:
      restartPolicy: Never
      containers:
      - name: pre-upgrade-backup-job
        image: "alpine"
        command: ["/bin/backup.sh"]
```
## Packaging and Sigining charts

Once you create your own chart. You can pack it and publish to a platform where it is easy for you to access.

```console
$ helm package ./nginx-chart
Sucessfully packaged chart and saved it to: 
 /home/documents/helm-charts/nginx-chart-0.1.0.tgz
```
The version after chart name comes from your Chart.yaml.

But you have no security, so we create a gpg key to authorize valid consumer for this package.

```console
 gpg --quick-generate-key "Sudhakar M"
About to create a key for:
    "Sudhakar M"

Continue? (Y/n) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /c/Users/sudhakar/.gnupg/trustdb.gpg: trustdb created
gpg: key E56E64645E7C75D1 marked as ultimately trusted
gpg: directory '/c/Users/sudhakar/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/c/Users/sudhakar/.gnupg/openpgp-revocs.d/381A7D966A2865F3EABFD46FE56E64645E7C75D1.rev'
public and secret key created and signed.

pub   rsa3072 2024-05-01 [SC] [expires: 2026-05-01]
      381A7D966A2865F3EABFD46FE56E64645E7C75D1
uid                      Sudhakar M
sub   rsa3072 2024-05-01 [E]

```

we package this again with key.
Refer: https://helm.sh/docs/topics/provenance/#the-workflow

```console
helm package --sign --key 'Sudhakar M' --keyring ~/.gnupg/secring.gpg ./nginx-chart

```
It  will create nginx-chart-0.1.0.tgz (a tar file of package) and nginx-chart-0.1.0.tgz.prov (a province file with signature)

you can verify the package and install everytime like this

`helm verify --keyring ./mypublickey ./nginx-0.1.0.tgz`

## Uploading Charts

The packaged chart we created in previous task which is signed. We should upload the same to common platform. Three things we need to prepare before uploading.

To prepare index.yaml file,
run this 
`helm repo index nginx-chart-files/ --url https://example.com/charts`

That generates an index.yaml file which contains the info about fescription and almost the data on Chart.yaml.

you can upload this to S3 bucket, Github, Google storage or any other common place.


1. Package file (nginx-0.1.0.tgz)
2. Index file (index.yaml)
3. province file (nginx-0.1.0.tgz.prov)



## End - Thank you!
{% endraw %} #To end the formatting for gh-pages. Do not remove this line

