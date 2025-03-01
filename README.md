# EICAR Test OCI Container

This guide provides step-by-step instructions to **build**, **annotate**, **push**, and **deploy** the `eicar-test-oci` container on **Docker.io** and deploy it in **Kubernetes** with Open Component Model (OCM) annotations.

---

## **1️⃣ Build the EICAR Test OCI Container**

### **1.1 Create the Dockerfile**
```dockerfile
# Use Ubuntu as the base image
FROM ubuntu:latest

# Define build-time arguments
ARG BUILD_DATE
ENV BUILD_DATE=${BUILD_DATE}

# Set metadata labels
LABEL maintainer="fafiorim"
LABEL opencomponentmodel.org/name="eicar-test"
LABEL opencomponentmodel.org/version="1.0.0"
LABEL opencomponentmodel.org/provider="fafiorim"
LABEL opencomponentmodel.org/repository="dockerhub"
LABEL opencomponentmodel.org/component="security-test"
LABEL opencomponentmodel.org/resourceType="container-image"
LABEL opencomponentmodel.org/system="linux"
LABEL opencomponentmodel.org/architecture="amd64"
LABEL opencomponentmodel.org/created="${BUILD_DATE}"
LABEL opencomponentmodel.org/license="MIT"
LABEL opencomponentmodel.org/source-url="https://github.com/fafiorim/eicar-test"
LABEL opencomponentmodel.org/maintainer="devops@fiorim.com"
LABEL opencomponentmodel.org/team="security-research"
LABEL opencomponentmodel.org/custom-tags="security,malware-test,eicar"
LABEL opencomponentmodel.org/deployment-mode="cloud-native, kubernetes-ready"
LABEL opencomponentmodel.org/compliance="ISO27001, GDPR, CCPA"
LABEL opencomponentmodel.org/release-stage="stable"
LABEL opencomponentmodel.org/support-end-date="2026-06-30"

# Install required dependencies (curl)
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Create the script to download EICAR test files
RUN echo "#!/bin/bash \n\
echo 'Downloading EICAR test files...' \n\
curl -o /app/eicar.txt https://secure.eicar.org/eicar.com.txt \n\
cat /app/eicar.txt \n\
curl -o /app/eicar.com https://secure.eicar.org/eicar.com \n\
cat /app/eicar.com \n\
curl -o /app/eicar_com.zip https://secure.eicar.org/eicar_com.zip \n\
curl -o /app/eicarcom2.zip https://secure.eicar.org/eicarcom2.zip \n\
echo 'EICAR files downloaded.'" > /app/download_eicar.sh

RUN chmod +x /app/download_eicar.sh

# Set entrypoint and command
ENTRYPOINT ["/bin/bash", "-c"]
CMD ["/app/download_eicar.sh && tail -f /dev/null"]
```

---

## **2️⃣ Build and Push the Container to Docker.io**

### **2.1 Build the Image**
```sh
docker build -t docker.io/fafiorim/eicar-test-oci:latest .
```

### **2.2 Push the Image to Docker.io**
```sh
docker push docker.io/fafiorim/eicar-test-oci:latest
```

### **2.3 Verify Image Annotations**
```sh
skopeo inspect --raw docker://docker.io/fafiorim/eicar-test-oci:latest | jq .
```
Ensure that **OCM annotations** are present under `"annotations"`.

---

## **3️⃣ Deploy to Kubernetes**

### **3.1 Create `deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eicar-test-oci
  labels:
    app: eicar-test-oci
spec:
  replicas: 1
  selector:
    matchLabels:
      app: eicar-test-oci
  template:
    metadata:
      labels:
        app: eicar-test-oci
    spec:
      containers:
        - name: eicar-test-oci
          image: docker.io/fafiorim/eicar-test-oci:latest
          imagePullPolicy: Always
          command: ["/bin/bash", "-c"]
          args: ["/app/download_eicar.sh && tail -f /dev/null"]
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

### **3.2 Apply Deployment to Kubernetes**
```sh
kubectl apply -f deployment.yaml
```

### **3.3 Verify Deployment**
Check the status of the pods:
```sh
kubectl get pods
```

Check the logs to ensure the container is running correctly:
```sh
kubectl logs -l app=eicar-test-oci
```

---

## **🎯 Summary**
1️⃣ **Built the `eicar-test-oci` container image** with EICAR test files.
2️⃣ **Annotated the image with OCM metadata.**
3️⃣ **Pushed the image to Docker.io.**
4️⃣ **Deployed the container to Kubernetes.**
5️⃣ **Verified the deployment and logs.**

Now your **EICAR test container is running successfully on Kubernetes!** 🚀

