# Clinic Release Lab

A DevOps portfolio lab that tests a containerized web app at three points:
directly in Docker, during a Kubernetes rollout, and through a Kubernetes Service.

## What I built

- Packaged a static clinic status page and a `/healthz` endpoint with Docker and Nginx.
- Built a GitHub Actions workflow that runs Docker smoke tests and deploys the
  Kubernetes manifest to a temporary kind cluster.
- Added an in-cluster smoke test that requests `/healthz` through the Kubernetes
  Service and checks that the response is `ok`.
- Published the image to a private Amazon ECR repository, deployed it to Amazon EKS
  by image digest, verified the Service from inside EKS, and tore down the AWS lab.

## Failure drills

| Scenario | What I observed | How I verified the fix or CI check |
| --- | --- | --- |
| Broken readiness probe | The replacement pod was running but stayed unready. Kubernetes events showed HTTP 404 because the probe used the wrong path. | I rolled back in kind and verified the Service returned `ok`. I reproduced the failure on a test branch and confirmed that the Kubernetes rollout check failed. |
| Broken Service selector | Docker and the Kubernetes rollout passed. The in-cluster request resolved the Service name but failed with `Connection refused` because the selector matched no pods. | I verified the failing Service check on PR #1 and closed it without merging. |

## Repository files

- `Dockerfile` packages the app in `site/`.
- `k8s/app.yaml` defines the Namespace, Deployment, and Service for kind.
- `.github/workflows/ci.yml` runs the Docker and Kubernetes checks.

CI reports failures on pull requests. Merge decisions were manual because branch
protection was unavailable under this private repository's plan.