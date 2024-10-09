
# My Tomcat App Helm Chart

This Helm chart deploys a Tomcat application with a blue-green deployment strategy. It supports environment-specific configurations for development, QA, and production environments.

## Prerequisites

- [Helm](https://helm.sh/docs/intro/install/) 3.x or higher
- [Kubernetes](https://kubernetes.io/docs/setup/) cluster

## Installation

### Step 1: Add the Helm Repository (if needed)

If the chart is hosted in a Helm repository, add the repository and update. If you're using a local chart, skip this step.

```bash
helm repo add my-repo https://charts.example.com
helm repo update
```

### Step 2: Install the Application

Install the application for the first time using the blue version:

```bash
helm install my-tomcat-app ./my-tomcat-app -f values-dev.yaml --set color=blue --namespace dev --create-namespace
```

- `my-tomcat-app`: The release name for the deployment.
- `./my-tomcat-app`: The path to the Helm chart (use the repository path if applicable).
- `-f values-dev.yaml`: Specifies the environment-specific values file.
- `--set color=blue`: Sets the initial deployment to blue.
- `--namespace dev`: Specifies the namespace for deployment.
- `--create-namespace`: Creates the namespace if it does not exist.

## Blue-Green Deployment Strategy

### Step 1: Deploy the Green Version

Deploy a new version (green) while keeping the blue version running:

```bash
helm upgrade my-tomcat-app ./my-tomcat-app -f values-dev.yaml --set color=green --namespace dev
```

### Step 2: Switch Traffic to the Green Version

Update the service to point to the green deployment by changing the service selector:

```bash
kubectl patch service my-tomcat-app -n dev -p '{"spec": {"selector": {"app.kubernetes.io/name": "my-tomcat-app", "app.kubernetes.io/instance": "my-tomcat-app-green"}}}'
```

## Rollback if the New Version Fails

If the green deployment encounters issues, roll back to the blue version as follows:

### Step 1: Switch Traffic Back to Blue

Revert the service selector to route traffic back to the blue deployment:

```bash
kubectl patch service my-tomcat-app -n dev -p '{"spec": {"selector": {"app.kubernetes.io/name": "my-tomcat-app", "app.kubernetes.io/instance": "my-tomcat-app-blue"}}}'
```

### Step 2: Rollback the Helm Release

Use the Helm rollback command to revert to the previous blue version:

```bash
helm rollback my-tomcat-app 1 --namespace dev
```

- `1` is the revision number for the blue deployment. You can list revisions using:

```bash
helm history my-tomcat-app --namespace dev
```

## Cleanup if the New Version Works

If the green version is stable and confirmed to work, follow these steps to remove the blue deployment.

### Step 1: Scale Down the Blue Deployment

Set the number of replicas for the blue deployment to zero:

```bash
kubectl scale deployment my-tomcat-app-blue --replicas=0 -n dev
```

### Step 2: Delete the Blue Deployment

Delete the old blue deployment:

```bash
kubectl delete deployment my-tomcat-app-blue -n dev
```

## Uninstall the Application

To completely remove the application from the Kubernetes cluster:

```bash
helm uninstall my-tomcat-app --namespace dev
```

## Summary of Commands

1. **Install Initial Version (Blue):**
   ```bash
   helm install my-tomcat-app ./my-tomcat-app -f values-dev.yaml --set color=blue --namespace dev --create-namespace
   ```

2. **Deploy New Version (Green):**
   ```bash
   helm upgrade my-tomcat-app ./my-tomcat-app -f values-dev.yaml --set color=green --namespace dev
   ```

3. **Switch Traffic to Green:**
   ```bash
   kubectl patch service my-tomcat-app -n dev -p '{"spec": {"selector": {"app.kubernetes.io/name": "my-tomcat-app", "app.kubernetes.io/instance": "my-tomcat-app-green"}}}'
   ```

4. **Rollback to Blue if Green Fails:**
   - Switch traffic back:
     ```bash
     kubectl patch service my-tomcat-app -n dev -p '{"spec": {"selector": {"app.kubernetes.io/name": "my-tomcat-app", "app.kubernetes.io/instance": "my-tomcat-app-blue"}}}'
     ```
   - Rollback Helm release:
     ```bash
     helm rollback my-tomcat-app 1 --namespace dev
     ```

5. **Remove Blue Deployment if Green Works:**
   - Scale down:
     ```bash
     kubectl scale deployment my-tomcat-app-blue --replicas=0 -n dev
     ```
   - Delete:
     ```bash
     kubectl delete deployment my-tomcat-app-blue -n dev
     ```

6. **Uninstall the Application (Optional):**
   ```bash
   helm uninstall my-tomcat-app --namespace dev
   ```

## Notes

- Ensure that `values-dev.yaml` is configured correctly for the deployment environment.
- Customize the commands for different environments (QA, production) by using the appropriate values file (`values-qa.yaml`, `values-prod.yaml`).
