# helm-webapp

**English** | [Nederlands](README.nl.md)

The example chart from the [Helm series on blog.stoiq.io](https://blog.stoiq.io/en/categories/helm/). A small web app ([podinfo](https://github.com/stefanprodan/podinfo)) packaged as a Helm chart, with everything the series covers: values, templates, named templates, a JSON schema, tests, a hook, a dependency and configuration with secrets.

## What's inside

```text
helm-webapp/
├── charts/webapp/                  ← the chart
│   ├── Chart.yaml                  ← with Valkey as a dependency
│   ├── Chart.lock
│   ├── values.yaml
│   ├── values.schema.json          ← validates your values
│   ├── templates/
│   │   ├── _helpers.tpl            ← names and labels
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml         ← with a checksum, restarts on new config
│   │   ├── ingress.yaml
│   │   ├── secret.yaml             ← token that persists via lookup
│   │   ├── service.yaml
│   │   ├── NOTES.txt
│   │   ├── hooks/migration-job.yaml
│   │   └── tests/test-connection.yaml
│   └── tests/                      ← unit tests for helm-unittest
├── examples/                       ← values for production, Ingress and Helmfile
└── .github/workflows/chart.yaml    ← test and publish
```

## What you need

- Helm 4 (Helm 3.8 or newer works too)
- A Kubernetes cluster, for example a local one with [kind](https://kind.sigs.k8s.io/)

## Install from the registry

```bash
helm install demo oci://ghcr.io/substoiq/charts/webapp --version 1.1.0 -n lab --create-namespace --wait
kubectl -n lab port-forward svc/demo-webapp 8080:80
```

Open <http://localhost:8080>.

Run the tests:

```bash
helm test demo -n lab --logs
```

## Install from this repository

```bash
git clone https://github.com/substoiq/helm-webapp.git
cd helm-webapp
helm repo add valkey https://valkey.io/valkey-helm/
helm dependency build charts/webapp
helm install demo charts/webapp -n lab --create-namespace --wait
```

`helm dependency build` needs the Valkey repository to be added first. `helm dependency update` doesn't.

## Settings

| Value | Default | What it does |
|:------|:--------|:-------------|
| `replicaCount` | `1` | Number of pods |
| `image.repository` | `ghcr.io/stefanprodan/podinfo` | Image of the app |
| `image.tag` | `""` | Empty = `appVersion` from `Chart.yaml` |
| `service.port` | `80` | Port of the Service |
| `ui.message` | `Hallo vanuit Helm` | Text shown in the app. May be a template, like `{{ .Release.Name }}` |
| `ui.color` | `#34577c` | Background colour, as `#rrggbb` |
| `resources` | requests and a memory limit | Leave out with `resources: null` |
| `extraEnv` | `{}` | Extra environment variables for podinfo |
| `nodeSelector` | `{}` | For example `kubernetes.io/arch: arm64` |
| `ingress.enabled` | `false` | Create an Ingress |
| `ingress.className` | `""` | For example `traefik` |
| `ingress.hosts` | `[]` | List of `host` with `paths` |
| `valkey.enabled` | `false` | Install Valkey alongside as a cache |
| `migration.enabled` | `true` | Example of a `pre-install`/`pre-upgrade` hook |
| `secret.existingSecret` | `""` | Use an existing Secret |
| `secret.apiToken` | `""` | Empty = the chart generates a token and keeps it |

A production example:

```bash
helm upgrade --install demo charts/webapp -n prod --create-namespace \
  -f examples/values-prod.yaml -f examples/values-ingress.yaml
```

## Check it yourself

```bash
helm lint --strict charts/webapp
helm template demo charts/webapp

helm plugin install https://github.com/helm-unittest/helm-unittest.git --verify=false
helm unittest charts/webapp
```

## Publishing

The workflow in `.github/workflows/chart.yaml` tests the chart on every push and pull request: lint, unit tests, an install in a kind cluster and `helm test`. Create a release with a tag like `webapp-v1.2.0` and it packages the chart, pushes it to GitHub Container Registry and signs it with cosign. Set `version` in `Chart.yaml` to the same number before tagging.

Verify the signature:

```bash
cosign verify ghcr.io/substoiq/charts/webapp:1.1.0 \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  --certificate-identity-regexp='^https://github\.com/substoiq/helm-webapp/.*$'
```

## The series

1. [What Helm is](https://blog.stoiq.io/en/posts/helm-what-is-helm/)
2. [Installing Helm](https://blog.stoiq.io/en/posts/helm-install/)
3. [Finding charts](https://blog.stoiq.io/en/posts/helm-find-charts/)
4. [The lifecycle of a release](https://blog.stoiq.io/en/posts/helm-release-lifecycle/)
5. [Values](https://blog.stoiq.io/en/posts/helm-values/)
6. [Cheat sheet](https://blog.stoiq.io/en/posts/helm-cheatsheet/)
7. [Your own chart](https://blog.stoiq.io/en/posts/helm-create-chart/)
8. [Templates](https://blog.stoiq.io/en/posts/helm-templates/)
9. [Helmify](https://blog.stoiq.io/en/posts/helm-helmify/)
10. [Flow control](https://blog.stoiq.io/en/posts/helm-flow-control/)
11. [Named templates](https://blog.stoiq.io/en/posts/helm-named-templates/)
12. [Debugging and testing](https://blog.stoiq.io/en/posts/helm-debug-test/)
13. [Dependencies](https://blog.stoiq.io/en/posts/helm-dependencies/)
14. [Hooks](https://blog.stoiq.io/en/posts/helm-hooks/)
15. [Packaging and publishing](https://blog.stoiq.io/en/posts/helm-package-publish/)
16. [Configuration and secrets](https://blog.stoiq.io/en/posts/helm-config-secrets/)
17. [CI/CD and GitOps](https://blog.stoiq.io/en/posts/helm-cicd-gitops/)
18. [Under the hood](https://blog.stoiq.io/en/posts/helm-under-the-hood/)

Comments inside the chart are in Dutch, as in the original series.

## License

MIT. See [LICENSE](LICENSE).
