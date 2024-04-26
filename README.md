# helm_practice

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