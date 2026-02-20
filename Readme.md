# Cherry-Pick Hotfix (Trunk-Based Development)

- In a trunk-based development model, main (trunk) is always releasable. Hotfixes are applied with minimal branching and selectively cherry-picked where required.
  
## This approach ensures

- Minimal divergence from main
- Fast production recovery
- Clean audit trail of fixes

## Step 1: Create a Release/Hotfix Branch from Production Commit

```bash
git checkout -b hotfix/v-1.0.1 <prod-commit-id> # commit-id from overlay kustomization.yaml
```

- Fix the bug in `main.go` - example Change the data in `line-24`

## push the changes to main branch

```bash
git add .
git commit -m "v-1.0.1"
git push origin hotfix/v-1.0.1
raise a pull request and merge to main - Note the commit-id of PR (not the main branch)
```

## Validate the Changes

```bash
git diff <prod-commit-id> <commit-id of PR> 
```

## Create a release tag

```bash
git tag -a v-1.0.0 -m "Hotfix: v-.10.1"
git push origin v-1.0.0
```

## Test locally

```bash
git checkout v-1.0.1
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    -t devops022/hello-api:58e8cfd \
    ./hello-api

docker push devops022/hello-api:58e8cfd
```

## Deploy the mainigests

```bash
Modify the mainfest in k8s/overlays/non-production/test/kustomization.yaml to add the tag 40e57cf
Deploy using: kustomize build k8s/overlays/non-production/test | kubectl apply -f -
```

## Test the changes

```bash
kubectl port-forward deployment/hello-api 8081:8080 -n apps 
curl -G 'http://localhost:8081?name=DevOps' 
```
