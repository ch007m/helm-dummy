## Instructions

To reproduce the problem, git clone this projct and execute this command:

```shell
helm template --dry-run --debug ./ \
  --set backstage.image.repository=my-backstage \
  --set backstage.image.tag=1.0
```

Helm will rendre then the image of the Deployment as such
```
install.go:178: [debug] Original chart version: ""
install.go:195: [debug] CHART PATH: /Users/cmoullia/temp/dummy

---
# Source: dummy/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-name-dummy
   ...
spec:
  replicas: 1
  ...
  template:
    ...
    spec:
      serviceAccountName: release-name-dummy
      securityContext:
        {}
      containers:
        - name: dummy
          securityContext:
            {}
          image: my-backstage1.0my-backstage:1.0
```
We can see that the image name populated `my-backstage1.0my-backstage:1.0` includes the 2 fields part of the `{{-required ....-}}` block
and not the return of the `{{ include "common.images.image" (dict "imageRoot" .Values.backstage.image "global" .Values.global) }}`

```text
{{/*
Check the required fields
*/}}
{{- define "backstage.image" -}}
{{- required "The repository name of the image is required (e.g. my-backstage:tag | docker.io/my-backstage:tag) !" .Values.backstage.image.repository -}}
{{- required "The image tag is required (e.g my-backstage:tag | docker.io/my-backstage:tag) !" .Values.backstage.image.tag -}}
{{ include "common.images.image" (dict "imageRoot" .Values.backstage.image "global" .Values.global) }}
{{- end -}}
```

