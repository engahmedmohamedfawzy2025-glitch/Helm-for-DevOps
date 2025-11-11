# ⚙️ Helm for DevOps Engineers – Complete & Advanced Guide

> **Author:** [ِAhmed Fawzy]
> **Level:** Beginner → Advanced
> **Goal:** A complete guide to mastering Helm for DevOps engineers, from fundamentals to CI/CD and GitOps integration.

---

## 📚 Table of Contents

1. [What is Helm](#-1-what-is-helm)
2. [Core Components](#-2-core-components)
3. [Installation](#-3-installation)
4. [Chart Structure](#-4-chart-structure)
5. [Chart.yaml](#-5-chartyaml)
6. [Values.yaml](#-6-valuesyaml)
7. [Templates](#-7-template-deploymentyaml)
8. [Install & Manage Releases](#-8-install-the-chart)
9. [Upgrade & Rollback](#-9-upgrade--rollback)
10. [Debugging](#-10-debugging)
11. [Repositories](#-11-adding-chart-repository)
12. [Templating Syntax](#-12-templating-syntax-quick-notes)
13. [_helpers.tpl](#-13-files-_helperstpl)
14. [Packaging & Distribution](#-14-packaging--distribution)
15. [Helm + CI/CD](#-15-helm--cicd-integration)
16. [Best Practices](#-16-helm-best-practices)
17. [Multi-Environment Deployment](#-17-multi-environment-deployment)
18. [Subcharts & Dependencies](#-18-subcharts--dependencies)
19. [Helm Hooks](#-19-helm-hooks-lifecycle-hooks)
20. [Helm Secrets](#-20-helm-secrets-managing-sensitive-data)
21. [Helmfile](#-21-helmfile--multi-environment-management)
22. [Helm + GitOps (ArgoCD)](#-22-helm--gitops-integration-argocd--flux)
23. [Linting & Testing](#-23-helm-linting--testing)
24. [Production Practices](#-24-helm-best-practices-production)
25. [Key Concepts](#-25-key-concepts-summary)
26. [CI/CD Example Pipeline](#-26-cicd-example-pipeline-jenkins)
27. [Common Commands](#-27-practical-demo-commands-summary)

---

## 🧠 1. What is Helm

Helm is a **package manager for Kubernetes**, similar to npm for Node.js or apt for Ubuntu.

It simplifies deploying, upgrading, and managing complex Kubernetes applications by packaging YAML manifests into reusable **charts**.

---

## 📦 2. Core Components

| Component      | Description                                                         |
| -------------- | ------------------------------------------------------------------- |
| **Chart**      | A package containing the application definition (YAML + templates). |
| **Repository** | A storage location for Helm charts (like Docker Hub).               |
| **Release**    | A running instance of a chart deployed in a cluster.                |
| **Values**     | Configuration values injected into templates.                       |

---

## ⚙️ 3. Installation

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

---

## 📁 4. Chart Structure

```bash
helm create myapp
```

Output:

```
myapp/
  ├── Chart.yaml
  ├── values.yaml
  ├── templates/
  ├── charts/
  └── .helmignore
```

---

## 🗾 5. Chart.yaml

```yaml
apiVersion: v2
name: myapp
description: A Helm chart for deploying a sample app
type: application
version: 0.1.0
appVersion: "1.0.0"
```

---

## 🧮 6. Values.yaml

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
```

---

## 🧱 7. Template (deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
```

---

## 🚀 8. Install the Chart

```bash
helm install myapp ./myapp
```

---

## 🔁 9. Upgrade & Rollback

```bash
helm upgrade myapp ./myapp
helm list
helm rollback myapp 1
```

---

## 🕵️‍♂️ 10. Debugging

```bash
helm template myapp ./myapp
helm get values myapp
```

---

## 🔗 11. Adding Chart Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-nginx bitnami/nginx
```

---

## 🧩 12. Templating Syntax Quick Notes

| Symbol                            | Meaning                 |
| --------------------------------- | ----------------------- |
| `{{ .Values.xxx }}`               | Access values.yaml      |
| `{{ .Release.Name }}`             | Release name            |
| `{{ .Chart.Name }}`               | Chart name              |
| `{{ include "template.name" . }}` | Include helper template |
| `{{- ... -}}`                     | Trim whitespace         |

---

## ⚙️ 13. Files `_helpers.tpl`

```yaml
{{- define "myapp.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end -}}
```

---

## 🪣 14. Packaging & Distribution

```bash
helm package myapp
helm push myapp-0.1.0.tgz oci://myrepo.azurecr.io/helm
```

---

## 🧰 15. Helm + CI/CD Integration

```bash
helm upgrade --install myapp ./helm-chart --namespace production
```

Commonly used in Jenkins or GitHub Actions after building and pushing Docker images.

---

## 🧠 16. Helm Best Practices

* Use environment-specific value files: `values-dev.yaml`, `values-prod.yaml`.
* Always run `helm lint` before deploying.
* Do not store secrets in `values.yaml`.
* Document every configurable value.
* Use dedicated namespaces for each environment.

---

## 🌍 17. Multi-Environment Deployment

```bash
helm install myapp -f values-dev.yaml ./myapp --namespace dev
helm install myapp -f values-prod.yaml ./myapp --namespace prod
```

---

## 🧩 18. Subcharts & Dependencies

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.2.0"
    repository: "https://charts.bitnami.com/bitnami"
```

```bash
helm dependency update
```

```yaml
# values.yaml
postgresql:
  auth:
    username: myuser
    password: mypass
```

---

## 🪝 19. Helm Hooks (Lifecycle Hooks)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: migrate-job
  annotations:
    "helm.sh/hook": post-install
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: myapp:latest
        command: ["python", "manage.py", "migrate"]
      restartPolicy: Never
```

---

## 🔐 20. Helm Secrets (Managing Sensitive Data)

```bash
helm plugin install https://github.com/jkroepke/helm-secrets
helm secrets install myapp ./myapp -f secrets.yaml
```

Supports SOPS, AWS KMS, GCP KMS, Azure KeyVault.

---

## 🧾 21. Helmfile – Multi-Environment Management

```yaml
repositories:
  - name: bitnami
    url: https://charts.bitnami.com/bitnami

releases:
  - name: myapp
    namespace: dev
    chart: ./helm/myapp
    values:
      - values-dev.yaml
```

```bash
helmfile sync
```

---

## 🧭 22. Helm + GitOps Integration (ArgoCD / Flux)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  source:
    repoURL: 'https://github.com/myorg/devops-infra'
    path: helm/myapp
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## 🧮 23. Helm Linting & Testing

```bash
helm lint ./myapp
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ .Release.Name }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: curl
    image: busybox
    command: ['wget']
    args: ['myapp:80']
  restartPolicy: Never
```

```bash
helm test myapp
```

---

## 🧱 24. Helm Best Practices (Production)

* Use `helm diff plugin` before upgrades.
* Store charts in private repositories.
* Modularize your charts.
* Validate YAML with `helm template` before deploy.
* Never commit secrets to Git.

---

## 📚 25. Key Concepts Summary

| Concept  | Meaning                      |
| -------- | ---------------------------- |
| Chart    | Packaged templates           |
| Release  | Deployed instance of a chart |
| Values   | Configurable inputs          |
| Template | YAML with dynamic logic      |
| Hook     | Lifecycle trigger            |
| Subchart | Nested dependency            |
| Helmfile | Multi-chart manager          |
| Secret   | Encrypted sensitive data     |

---

## 🧰 26. CI/CD Example Pipeline (Jenkins)

```groovy
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'docker build -t myapp:latest .'
        sh 'docker push myrepo/myapp:latest'
      }
    }
    stage('Deploy') {
      steps {
        sh 'helm upgrade --install myapp ./helm/myapp -f values-prod.yaml'
      }
    }
  }
}
```

---

## 🚀 27. Practical Demo Commands Summary

```bash
helm create myapp
helm install myapp ./myapp
helm upgrade myapp ./myapp
helm uninstall myapp
helm get values myapp
helm template myapp ./myapp
helm lint ./myapp
```
