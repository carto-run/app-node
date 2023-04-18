# backstage

## Creating the Workload

```
tanzu apps workload create backstage \
  --namespace dev \
  --git-branch master \
  --git-repo https://github.com/backstage/backstage \
  --label apps.tanzu.vmware.com/has-tests=true \
  --label app.kubernetes.io/part-of=backstage \
  --param-yaml testing_pipeline_matching_labels='{"apps.tanzu.vmware.com/pipeline":"noop-pipeline"}' \
  --param-yaml testing_pipeline_params='{}' \
  --type web \
  --yes
```

## Logs

```
tanzu apps workload tail backstage
```

## Configuration

<table>

<tr>
<th> Item </th>
<th> Config </th>
</tr>

<tr>
<td> Scan Policy </td>
<td> 
  
[default](resources/scan-policy.yaml)
  
</td>
</tr>

<tr>
<td> Pipeline </td>
<td>

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  labels:
    apps.tanzu.vmware.com/pipeline: noop-pipeline
  name: developer-defined-noop-tekton-pipeline
  namespace: dev
spec:
  params:
  - name: source-url
    type: string
  - name: source-revision
    type: string
  tasks:
  - name: noop-task
    params:
    - name: source-url
      value: $(params.source-url)
    - name: source-revision
      value: $(params.source-revision)
    taskSpec:
      metadata: {}
      params:
      - name: source-url
        type: string
      - name: source-revision
        type: string
      steps:
      - image: bash
        name: noop
        resources: {}
        script: |
          echo "Nothing to do for $(params.source-url)/$(params.source-revision)"
```  

</td>
</tr>

<tr>
<td> Supply Chain </td>
<td> source-test-scan-to-url </td>
</tr>

</table>
