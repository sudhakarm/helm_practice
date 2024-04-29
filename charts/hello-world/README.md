Create a directory called templates, it should contain all the yaml definitions with place holders.

It must be a replacable and reusable piece of block in every definition to form a template. We have to keep that in mind while writing the definition.

Below is the directory structure for helm chart.
- templates (dir)
    - pod.yaml
    - service.yaml
- Values.yaml
- Chart.yaml

## Chart.yaml
Helm 2 works with `apiVersion: v1`
Helm 3 works with `apiVersion: v2`

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

