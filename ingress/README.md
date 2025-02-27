## Install Nginx Ingress Controller
```
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

## Create Nginx Deployments with Custom Content
### app1.yml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        # Add custom content
        command: ["/bin/sh", "-c"]
        args:
          - echo "<h1>App 1 - Nginx</h1>" > /usr/share/nginx/html/index.html;
            exec nginx -g 'daemon off;'
---
apiVersion: v1
kind: Service
metadata:
  name: app1-svc
spec:
  selector:
    app: app1
  ports:
    - port: 80
      targetPort: 80
```

### app2.yml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        # Add custom content
        command: ["/bin/sh", "-c"]
        args:
          - echo "<h1>App 2 - Nginx</h1>" > /usr/share/nginx/html/index.html;
            exec nginx -g 'daemon off;'
---
apiVersion: v1
kind: Service
metadata:
  name: app2-svc
spec:
  selector:
    app: app2
  ports:
    - port: 80
      targetPort: 80
```

### Apply the configurations:
```
kubectl apply -f app1.yaml -f app2.yaml
```

## Create Ingress Rules

### ingress.yml:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-svc
            port:
              number: 80
```

### Apply the ingress:
```
kubectl apply -f ingress.yml
```

### Get the ingress IP:
```
kubectl get ingress
```

### Access the services:
```
curl http://<INGRESS_IP>/app1
curl http://<INGRESS_IP>/app2
```

### Check controller logs:
```
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
```

### Verify endpoints:
```
kubectl get endpoints app1-svc app2-svc
```
