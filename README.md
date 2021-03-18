# tektondemo

1. Run the following command to install Tekton Pipelines and its dependencies:

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

2. Apply the created Tasks

```
kc apply -f 1-task.yaml
```

```
kc apply -f 2-task.yaml
```

3. Apply the build-Resources

```
kc apply -f 3-build-resources.yaml
```

4. Apply the 4-pipeline.yaml

```
kc apply -f 4-pipeline.yaml
```

5. Apply the 5-pipelineRun.yaml

```
kc apply -f 5-pipelineRun.yaml
```

6. Deployment and Service files will be applied during pipeline run.

7. For 8-ingress.yaml you can apply it manually or you can modify 2-task.yaml file to make pipeline deploy it for you.
