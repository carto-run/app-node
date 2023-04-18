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
  --param scanning_source_policy=allow-everything-scan-policy \
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
 
```yaml
---
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanPolicy
metadata:
  name: allow-everything-scan-policy
  namespace: dev
  labels:
    'app.kubernetes.io/part-of': 'enable-in-gui'
spec:
  regoFile: |
    package main

    # Accepted Values: "Critical", "High", "Medium", "Low", "Negligible", "UnknownSeverity"
    notAllowedSeverities := ["UnknownSeverity"]
    ignoreCves := []

    contains(array, elem) = true {
      array[_] = elem
    } else = false { true }

    isSafe(match) {
      severities := { e | e := match.ratings.rating.severity } | { e | e := match.ratings.rating[_].severity }
      some i
      fails := contains(notAllowedSeverities, severities[i])
      not fails
    }

    isSafe(match) {
      ignore := contains(ignoreCves, match.id)
      ignore
    }

    deny[msg] {
      comps := { e | e := input.bom.components.component } | { e | e := input.bom.components.component[_] }
      some i
      comp := comps[i]
      vulns := { e | e := comp.vulnerabilities.vulnerability } | { e | e := comp.vulnerabilities.vulnerability[_] }
      some j
      vuln := vulns[j]
      ratings := { e | e := vuln.ratings.rating.severity } | { e | e := vuln.ratings.rating[_].severity }
      not isSafe(vuln)
      msg = sprintf("CVE %s %s %s", [comp.name, vuln.id, ratings])
    }
``` 
  
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
