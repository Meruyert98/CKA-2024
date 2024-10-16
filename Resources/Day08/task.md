## Task 8/40


In this exercise, you will create a Deployment with multiple replicas. After inspecting the Deployment, you will update its Pod template. You will also be able to use the rollout history to roll back to a previous revision.

> [!NOTE]
> If you do not already have a Kubernetes cluster, you can create a local Kubernetes cluster by following [Day06 Video](https://youtu.be/RORhczcOrWs)


## Replicaset
- Create a new Replicaset based on the nginx image with 3 replicas
  ```yaml
  apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    env: demo
spec:
  template:
    metadata:
      name: nginx-pod
      labels:
        env: demo
        type: frontend
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
          - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: demo
  ```
- Update the replicas to 4 from the YAML
```sh
kubectl edit rs
# then in the replicas line Shift+A replicas: 4 press ESC then type :wq!
kubectl get pods
# Pods created but yaml file still shows 3
```
- Update the replicas to 6 from the command line
```
kubectl scale --replicas=6 rs/nginx-rs
kubectl get pods
# Shows 6 replicas
```
## Deployment
1. Create a Deployment named `nginx` with 3 replicas. The Pods should use the `nginx:1.23.0` image and the name `nginx`. The Deployment uses the label `tier=backend`. The Pod template should use the label `app=v1`.
   ```yaml
   apiVersion: apps/v1  # Correct apiVersion for Deployment
kind: Deployment
metadata:
  name: nginx-deployment  # Changed name to reflect it's a deployment
  labels:
    tier: backend
spec:
  replicas: 3  # Number of pod replicas
  selector:
    matchLabels:
      tier: backend  # Selector to match pods with the same label
  template:
    metadata:
      labels:
        tier: backend  # Match the selector with the same label
        app: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.0
```
2. List the Deployment and ensure the correct number of replicas is running.
```
kubectl apply -f pod_deploy.yaml
kubectl get pods
```
4. Update the image to `nginx:1.23.4`.
```
kubectl set image deploy/nginx-deployment nginx=nginx:1.23.4
```
5. Verify that the change has been rolled out to all replicas.
```
kubectl rollout history deployment/nginx-deployment
```
6. Assign the change cause "Pick up patch version" to the revision.
```sh
kubectl set image deploy/nginx-deployment nginx=nginx:1.23.4 --record
kubectl rollout history deployment/nginx-deployment

# or new version

kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Pick up patch version"
kubectl rollout history deployment/nginx-deployment
```

7. Scale the Deployment to 5 replicas.
```
kubectl scale --replicas=5 deploy/nginx-deployment
```
8. Have a look at the Deployment rollout history.
```
kubectl rollout history deploy/nginx-deployment
```
9. Revert the Deployment to revision 1.
```
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```
10. Ensure that the Pods use the image `nginx:1.23.0`.
```
kubectl get deployment nginx-deployment -o=jsonpath='{.spec.template.spec.containers[0].image}
```

## Troubleshooting the issue
1. Apply the below YAML and fix the issue with it

``` YAML
apiVersion: v1
kind:  Deployment
metadata:
  name: nginx-deploy
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: demo
```

2. Apply the below YAML and fix the issue with it

``` YAML
apiVersion: v1
kind:  Deployment
metadata:
  name: nginx-deploy
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: dev
```
3. **Share your learnings**: Document your key takeaways and insights in a blog post and social media update
4. **Make it public**: Share what you learn publicly on LinkedIn or Twitter.
   - **Tag us and use the hashtag**: Include the following in your post:
     - Tag [@PiyushSachdeva](https://www.linkedin.com/in/piyush-sachdeva) and [@CloudOps Community](https://www.linkedin.com/company/thecloudopscomm) (on both platforms)
     - Use the hashtag **#40daysofkubernetes**
     - **Embed the video**: Enhance your blog post by embedding the video lesson from the Kubernetes series. This will provide visual context and reinforce your written explanations.

## Blog Post Focus 📝

- **Clarity is essential**: Write your blog post clearly and concisely, making it easy for anyone to grasp the concepts, regardless of their prior Kubernetes experience.

