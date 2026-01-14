docker run --name postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres

Building a Kubernetes CRD controller in Python is typically done using frameworks like **Kopf** or the **[Operator SDK]([https://sdk.operatorframework.io/)**](https://sdk.operatorframework.io/\)** "https://sdk.operatorframework.io/)**") (which supports Python). The core of the controller is the reconciliation loop, a function that runs continuously to ensure the cluster's actual state matches the desired state defined in your Custom Resource (CR). 

Core Concepts

- **Custom Resource Definition (CRD):** Extends the Kubernetes API with a new resource type.  
- **Custom Resource (CR):** An instance of a CRD, which holds the desired configuration.  
- **Controller:** Software that watches for changes to CRs and executes logic.  
- **Reconciliation Loop:** The function within the controller that compares the `desired state` (from the CR's spec) with the `actual state` in the cluster and takes necessary actions (create, update, or delete other Kubernetes resources like Pods, Services, etc.). 

Example using Kopf (Python Operator Framework)

Kopf is a popular and relatively simple Python framework for building operators. The primary logic is contained within a handler function decorated with `@kopf.on.create`, `@kopf.on.update`, or `@kopf.on.delete`. 

1. Define the CRD (Example: `website.yaml`) 

This YAML defines a simple `Website` custom resource. 

yaml

```  
apiVersion: stable.example.com/v1  
kind: CustomResourceDefinition  
metadata:  
  name: websites.stable.example.com  
spec:  
  group: stable.example.com  
  versions:  
    - name: v1  
      served: true  
      storage: true  
      schema:  
        openAPIV3Schema:  
          type: object  
          properties:  
            spec:  
              type: object  
              properties:  
                image:  
                  type: string  
                  description: The Docker image for the web server.  
                replicas:  
                  type: integer  
                  description: The number of desired replicas.  
  scope: Namespaced  
  names:  
    plural: websites  
    singular: website  
    kind: Website  
    shortNames:  
    - web  
```

2. Implement the Controller with Reconciliation Logic (`operator.py`) 

This Python code uses Kopf to watch for `Website` CRs and reconcile them into a Kubernetes `Deployment` and `Service`. 

python

```  
import kopf  
import kubernetes.client  
import yaml

@kopf.on.create('stable.example.com', 'v1', 'websites')  
@kopf.on.update('stable.example.com', 'v1', 'websites')  
def reconcile_website(spec, meta, logger, **kwargs):  
    # Desired state from the Custom Resource (CR) spec  
    image = spec.get('image')  
    replicas = spec.get('replicas')  
    name = meta.get('name')  
    namespace = meta.get('namespace')

    if not image or not replicas:  
        raise kopf.PermanentError(f"Image and replicas must be specified in Spec.")

    logger.info(f"Reconciling Website: {name} (Image: {image}, Replicas: {replicas})")

    # Define the desired Deployment  
    deployment_yaml = f"""  
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: {name}-deployment  
  ownerReferences:  
    - apiVersion: stable.example.com/v1  
      kind: Website  
      name: {name}  
      uid: {meta['uid']}  
spec:  
  selector:  
    matchLabels:  
      app: {name}  
  replicas: {replicas}  
  template:  
    metadata:  
      labels:  
        app: {name}  
    spec:  
      containers:  
      - name: webserver  
        image: {image}  
        ports:  
        - containerPort: 80  
"""  
    deployment_body = yaml.safe_load(deployment_yaml)

    # Use Kubernetes Python client to apply the desired state  
    api_apps = kubernetes.client.AppsV1Api()  
    try:  
        # Create or update the Deployment  
        api_apps.create_namespaced_deployment(  
            body=deployment_body,  
            namespace=namespace,  
        )  
        logger.info(f"Deployment {name}-deployment created/updated.")  
    except kubernetes.client.ApiException as e:  
        # Handle update if it already exists (basic handling)  
        if e.status == 409:  
            api_apps.replace_namespaced_deployment(  
                name=f"{name}-deployment",  
                body=deployment_body,  
                namespace=namespace,  
            )  
            logger.info(f"Deployment {name}-deployment replaced.")  
        else:  
            raise kopf.PermanentError(f"Failed to manage deployment: {e}")

    # The function returns the actual state (status) which will be updated in the CR  
    return {'deployment_name': f'{name}-deployment', 'replicas_set': replicas}

@kopf.on.delete('stable.example.com', 'v1', 'websites')  
def delete_website(meta, logger, **kwargs):  
    # Kubernetes garbage collection handles deleting the owned Deployment (ownerReferences field is key)  
    logger.info(f"Deletion request for Website: {meta.get('name')}. Owned resources will be garbage collected.")  
    return {'message': 'Website resource deleted.'}

```

3. How to Run

4. **Install Kopf:** `pip install kopf kubernetes`  
5. **Apply the CRD:** `kubectl apply -f website.yaml`  
6. **Run the operator (locally for testing):** `kopf run operator.py --namespace <your-namespace>` 

When you create a `Website` custom resource (e.g., `kubectl apply -f my-web-app-cr.yaml`), the `reconcile_website` function is triggered, which in turn creates a `Deployment` based on your specifications.