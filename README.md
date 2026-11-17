# helm-webapp

De voorbeeldchart uit de [Helm-serie op blog.stoiq.io](https://blog.stoiq.io/categories/helm/). Een kleine webapp ([podinfo](https://github.com/stefanprodan/podinfo)) als Helm-chart, met alles wat de serie behandelt: values, templates, named templates, een JSON-schema, tests, een hook, een afhankelijkheid en configuratie met secrets.

*English summary below.*

## Wat erin zit

```text
helm-webapp/
├── charts/webapp/                  ← de chart
│   ├── Chart.yaml                  ← met Valkey als afhankelijkheid
│   ├── Chart.lock
│   ├── values.yaml
│   ├── values.schema.json          ← controleert je values
│   ├── templates/
│   │   ├── _helpers.tpl            ← namen en labels
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml         ← met checksum, herstart bij nieuwe config
│   │   ├── ingress.yaml
│   │   ├── secret.yaml             ← token dat blijft met lookup
│   │   ├── service.yaml
│   │   ├── NOTES.txt
│   │   ├── hooks/migration-job.yaml
│   │   └── tests/test-connection.yaml
│   └── tests/                      ← unit tests voor helm-unittest
├── examples/                       ← values voor productie, Ingress en Helmfile
└── .github/workflows/chart.yaml    ← testen en publiceren
```

## Wat je nodig hebt

- Helm 4 (Helm 3.8 of nieuwer werkt ook)
- Een Kubernetes-cluster, bijvoorbeeld lokaal met [kind](https://kind.sigs.k8s.io/)

## Installeren vanuit deze repository

```bash
git clone https://github.com/substoiq/helm-webapp.git
cd helm-webapp
helm dependency build charts/webapp
helm install demo charts/webapp -n lab --create-namespace --wait
kubectl -n lab port-forward svc/demo-webapp 8080:80
```

Open <http://localhost:8080>.

Testen:

```bash
helm test demo -n lab --logs
```

## Installeren vanuit de registry

```bash
helm install demo oci://ghcr.io/substoiq/charts/webapp --version 1.1.0 -n lab --create-namespace
```

## Instellingen

| Value | Standaard | Wat het doet |
|:------|:----------|:-------------|
| `replicaCount` | `1` | Aantal pods |
| `image.repository` | `ghcr.io/stefanprodan/podinfo` | Image van de app |
| `image.tag` | `""` | Leeg = `appVersion` uit `Chart.yaml` |
| `service.port` | `80` | Poort van de Service |
| `ui.message` | `Hallo vanuit Helm` | Tekst in de app. Mag een sjabloon zijn, zoals `{{ .Release.Name }}` |
| `ui.color` | `#34577c` | Achtergrondkleur, als `#rrggbb` |
| `resources` | requests en een geheugenlimiet | Laat leeg met `resources: null` |
| `extraEnv` | `{}` | Extra omgevingsvariabelen voor podinfo |
| `nodeSelector` | `{}` | Bijvoorbeeld `kubernetes.io/arch: arm64` |
| `ingress.enabled` | `false` | Een Ingress aanmaken |
| `ingress.className` | `""` | Bijvoorbeeld `traefik` |
| `ingress.hosts` | `[]` | Lijst van `host` met `paths` |
| `valkey.enabled` | `false` | Valkey meeinstalleren als cache |
| `migration.enabled` | `true` | Voorbeeld van een `pre-install`/`pre-upgrade`-hook |
| `secret.existingSecret` | `""` | Een bestaand Secret gebruiken |
| `secret.apiToken` | `""` | Leeg = de chart maakt een token en bewaart het |

Een voorbeeld voor productie:

```bash
helm upgrade --install demo charts/webapp -n prod --create-namespace \
  -f examples/values-prod.yaml -f examples/values-ingress.yaml
```

## Zelf controleren

```bash
helm lint --strict charts/webapp
helm template demo charts/webapp

helm plugin install https://github.com/helm-unittest/helm-unittest.git --verify=false
helm unittest charts/webapp
```

## Publiceren

De workflow in `.github/workflows/chart.yaml` test de chart bij elke pull request. Push je een tag als `webapp-v1.1.0`, dan pakt hij de chart in, zet hem in GitHub Container Registry en ondertekent hem met cosign. Zet de `version` in `Chart.yaml` vóór de tag op hetzelfde nummer.

## De serie

1. [Wat Helm is](https://blog.stoiq.io/posts/helm-what-is-helm/)
2. [Installeren](https://blog.stoiq.io/posts/helm-install/)
3. [Charts vinden](https://blog.stoiq.io/posts/helm-find-charts/)
4. [De levenscyclus van een release](https://blog.stoiq.io/posts/helm-release-lifecycle/)
5. [Values](https://blog.stoiq.io/posts/helm-values/)
6. [Spiekbrief](https://blog.stoiq.io/posts/helm-cheatsheet/)
7. [Je eigen chart](https://blog.stoiq.io/posts/helm-create-chart/)
8. [Templates](https://blog.stoiq.io/posts/helm-templates/)
9. [Helmify](https://blog.stoiq.io/posts/helm-helmify/)
10. [Flow control](https://blog.stoiq.io/posts/helm-flow-control/)
11. [Named templates](https://blog.stoiq.io/posts/helm-named-templates/)
12. [Debuggen en testen](https://blog.stoiq.io/posts/helm-debug-test/)
13. [Dependencies](https://blog.stoiq.io/posts/helm-dependencies/)
14. [Hooks](https://blog.stoiq.io/posts/helm-hooks/)
15. [Packagen en publiceren](https://blog.stoiq.io/posts/helm-package-publish/)
16. [Configuratie en secrets](https://blog.stoiq.io/posts/helm-config-secrets/)
17. [CI/CD en GitOps](https://blog.stoiq.io/posts/helm-cicd-gitops/)
18. [Onder de motorkap](https://blog.stoiq.io/posts/helm-under-the-hood/)

## English summary

The example chart from the [Helm series on blog.stoiq.io](https://blog.stoiq.io/en/categories/helm/). It packages [podinfo](https://github.com/stefanprodan/podinfo) and shows values, templates, named templates, a JSON schema, tests, a hook, a dependency (Valkey) and configuration with secrets.

```bash
helm dependency build charts/webapp
helm install demo charts/webapp -n lab --create-namespace --wait
helm test demo -n lab --logs
```

Comments in the chart are in Dutch; the series is available in English at [blog.stoiq.io/en](https://blog.stoiq.io/en/).

## Licentie

MIT. Zie [LICENSE](LICENSE).
