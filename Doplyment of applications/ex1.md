EX1. Create a Ubuntu docker image pod on k8s (k8s + docker&cri + flannel network plugin)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  containers:
  - name: ubuntu
    image: ubuntu:latest
    command: ["/bin/sleep", "3650d"] #this can help k8s to create the pod
    imagePullPolicy: IfNotPresent
#Some options:    
#     volumeMounts:
#     - name: config-volume
#       mountPath: /home/config.txt
#       subPath: example-config
#   restartPolicy: Always
#   volumes:
#   - name: config-volume
#     configMap:
#       name: example-config
```