Here's how to **create a Dockerfile for a simple web application** and **deploy it on a Kubernetes (K8s) cluster**. We'll use a basic **Node.js + Express** application for this example.

---

## **1. Simple Web App Code (Node.js)**

Create a directory `simple-webapp/` and inside it, create:

### `app.js`

```js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello from the simple web app running in Kubernetes!');
});

app.listen(port, () => {
  console.log(`App listening at http://localhost:${port}`);
});
```

### `package.json`

```json
{
  "name": "simple-webapp",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

## **2. Dockerfile**

Create a `Dockerfile` in the same directory:

```Dockerfile
# Stage 1: Build
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

---

## **3. Build and Push Docker Image**

Replace `<your-dockerhub-username>` with your Docker Hub username:

```bash
docker build -t <your-dockerhub-username>/simple-webapp:latest .
docker push <your-dockerhub-username>/simple-webapp:latest
```

---

## **4. Kubernetes Deployment YAML**

Create `k8s-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-webapp
  template:
    metadata:
      labels:
        app: simple-webapp
    spec:
      containers:
      - name: simple-webapp
        image: <your-dockerhub-username>/simple-webapp:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp-service
spec:
  type: LoadBalancer
  selector:
    app: simple-webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```

---

## **5. Deploy on Kubernetes**

Make sure you're connected to your Kubernetes cluster (`kubectl config use-context your-cluster`), then apply the YAML:

```bash
kubectl apply -f k8s-deployment.yaml
```

---

## **6. Verify Deployment**

```bash
kubectl get pods
kubectl get svc
```

Wait for the `EXTERNAL-IP` of `simple-webapp-service` and open it in your browser.

--
