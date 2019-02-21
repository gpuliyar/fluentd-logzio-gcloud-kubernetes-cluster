# fluentd-logzio-gcloud-kubernetes-cluster
Configure Logzio-Fluentd configuration and deploy it on the GCloud Kubernetes Engine


## What is Project for?
The project will show how to use the Logzio and Fluentd to collect the logs in your Google Cloud Kubernetes Cluster. 

## Prerequisite
* You should have some basic working knowledge to create clusters in Google Cloud
* You should know what Kubernetes is.
* You should know what is a Deamonset in Kubernetes.

> You are trying to aggregate your log and keep it in a central repository. Multiple frameworks can aggregate the Logs, and numerous vendors in the market allow collecting the Logs and manage in a central repository. For my use-case, I chose fluentd for log aggregation and Logz.io to handle logs. If your use-case is the same, then you landed on the right page.

## Check out the below
Take a look at the Github URL where fluentd manages its configuration. I'm going to reuse what they built and add additional commands that can help you to set up the feature and verify them quick. https://github.com/logzio/logzio-k8s/

> Note: I'm using Google Console Shell to execute the commands. You can either use that or the Google Cloud SDK.

## First, create a Cluster 
```
gcloud container clusters create fluentd-logzio-cluster --zone us-central1-a --disk-size=20 --disk-type=pd-standard
```

## Second, find your account info. If you already know, then skip this section
```
gcloud info | grep Account
```

## Third, give Cluster Admin privilege
```
kubectl create clusterrolebinding <give a name to your binding> --clusterrole=cluster-admin --user=<provide the account info that you found from the previous command>
```

## Now, create a new file with the below configuration (name file as fluentd-logzio-daemonset.yaml). And save the file.
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-system
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd-logzio
  namespace: kube-system
  labels:
    k8s-app: fluentd-logzio
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  template:
    metadata:
      labels:
        k8s-app: fluentd-logzio
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: logzio/logzio-k8s:1.0.3
        env:
        - name:  LOGZIO_TOKEN
          value: "jOfZnYHaWzKzqeaWLiECYCdMaVulKuiP"
        - name:  LOGZIO_URL
          value: "https://listener.logz.io:8071"
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
          ---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-system
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd-logzio
  namespace: kube-system
  labels:
    k8s-app: fluentd-logzio
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  template:
    metadata:
      labels:
        k8s-app: fluentd-logzio
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: logzio/logzio-k8s:1.0.3
        env:
        - name:  LOGZIO_TOKEN
          value: "< configure the token that you got from logz.io >"
        - name:  LOGZIO_URL
          value: "< configure the logz.io endpoint url here >"
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Finally, deploy the Daemonset
```
kubectl create -f fluentd-logzio-daemonset.yaml
```

> Voila, you are done. Your setup is complete

## Check the logz.io portal to see your logs (possibly after 2 to 3 minutes)
