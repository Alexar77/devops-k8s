# Container Registry (github packages)

## Github Packages
* create personal access token (settings --> Developer settings -- > Personal Access Tokens), select classic
* select read:packages
* save the token to a file
* to see packages, go to your github profile and select tab Packages
* tag an image
```bash
docker build -t ghcr.io/<GITHUB-USERNAME>/ds-spring:latest -f nonroot.Dockerfile .
```
* login to docker registry
```bash
cat ~/github-image-repo.txt | docker login ghcr.io -u <GITHUB-USERNAME> --password-stdin
```
* push image
```bash
docker push ghcr.io/<GITHUB-USERNAME>/ds-spring:latest
```
## create docker login secret

* create a .dockerconfigjson file, like this
```json
{
    "auths": {
        "https://ghcr.io":{
            "username":"REGISTRY_USERNAME",
            "password":"REGISTRY_TOKEN",
            "email":"REGISTRY_EMAIL",
            "auth":"BASE_64_BASIC_AUTH_CREDENTIALS"
    	}
    }
}
```


* create <BASE_64_BASIC_AUTH_CREDENTIALS> from the command
```bash
echo -n <USER>:<TOKEN> | base64
```

## create dockercongig secret
```bash
kubectl create secret docker-registry registry-credentials --from-file=.dockerconfigjson=.dockerconfig.json
```


# Deployment

## minikube

```bash
minikube addons enable ingress
```





# Links
* [Spring boot health probes](https://www.baeldung.com/spring-liveness-readiness-probes)
