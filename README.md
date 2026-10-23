> **Part of the [QorVertex live demo](https://qorvertex.com).** A small sample application that shows a shared PostgreSQL database: one repository owns the schema, others read and write it, and a deployment repository wires them together. It is intentionally minimal and not production code.

# acme-orders-deploy

Runtime wiring for the Acme order system. **No application code lives here** —
the services are built from three sibling repositories:

| compose service | built from | role |
| --- | --- | --- |
| `migrate` | [acme-orders-schema](https://github.com/qorvertex-demo/acme-orders-schema) | applies migrations, then exits |
| `orders-api` | [acme-orders-api](https://github.com/qorvertex-demo/acme-orders-api) | HTTP order intake |
| `reporting` | [acme-orders-reporting](https://github.com/qorvertex-demo/acme-orders-reporting) | reads the database, calls the API |

All three point at the same `orders-db` instance, and `reporting` reaches
`orders-api` over HTTP via `ORDERS_API_URL`.

```bash
docker compose up --build
```

## Kubernetes

The same three services, wired for a cluster (`k8s/`):

| manifest | kind | built from |
| --- | --- | --- |
| `migrate-job.yaml` | Job (ArgoCD PreSync hook) | acme-orders-schema |
| `orders-api.yaml` | Deployment + Service | acme-orders-api |
| `reporting.yaml` | CronJob | acme-orders-reporting |
| `ingress.yaml` | Ingress | (routes to `orders-api`) |

`reporting` reaches the API through the in-cluster Service via
`ORDERS_API_URL=http://orders-api:4010`.

`ingress.yaml` publishes the API at `orders.acme.internal` — the routing is the
only record of which repository answers a public URL, since the image it names
is built in `acme-orders-api`.

```bash
kubectl apply -f k8s/
```
