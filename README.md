# NGINX Plus Container

This repository demonstrates the process to create an NGINX Plus container.

There are two variants:

- NGINX Plus
- NGINX Plus with the NGINX Agent

Assumptions:

- You have an NGINX Plus trial or subscription
- You have a 1Password subscription (if using for secrets management)
- You have shell access (Bash or ZSH) and docker

**_NOTE:_** To build an NGINX Plus container, you will need access to the NGINX Plus repository.  Download the repository certificate and key from [MyF5](https://my.f5.com) and name them *nginx-repo.crt* and *nginx-repo.key*.

**_NOTE:_** If you are leveraging 1Password for secrets management, you will need to replace my example 1Password secret reference URIs with your own URIs. 

**_NOTE:_** The code provided here is for demonstration purposes and should not be used in production environments.

## NGINX Plus

### Build an NGINX Plus Container

The commands below will build a basic NGINX Plus container:

```bash
cd nginx-plus
docker build --no-cache \
--secret id=nginx-key,src=nginx-repo.key \
--secret id=nginx-crt,src=nginx-repo.crt \
-t nginxplus --load .
```

Or if you're using 1Password to store the certificate and key:

```bash
cd nginx-plus
export NGINX_CRT="op://Work/nginx-repo-crt/nginx-repo.crt"
export NGINX_KEY="op://Work/nginx-repo-key/nginx-repo.key"
op run -- docker build --no-cache \
--secret id=nginx-key,env=NGINX_KEY \
--secret id=nginx-crt,env=NGINX_CRT \
-t nginxplus --load .
```

### Run an NGINX Plus Container

To run the NGINX Plus container, execute the following command:

```bash
docker run --name nginx-plus -d \
    -p 80 \
    -p 443 \
    nginxplus
```

## Build NGINX Plus Container w/ Agent

The commands below will build a basic NGINX Plus container with the NGINX Agent:

```bash
cd nginx-plus-agent
docker build  --no-cache \
--secret id=nginx-crt,src=nginx-repo.crt \
--secret id=nginx-key,src=nginx-repo.key \
--secret id=agent-crt,src=agent.crt \
--secret id=agent-key,src=agent.key \
-t nginx_plus_agent --load .
```

Or if you're using 1Password to store the certificate and key:

```bash
cd nginx-plus-agent
export NGINX_CRT="op://Work/nginx-repo-crt/nginx-repo.crt"
export NGINX_KEY="op://Work/nginx-repo-key/nginx-repo.key"
export AGENT_CRT="op://Work/agent-crt/agent.crt"
export AGENT_KEY="op://Work/agent-key/agent.key"
op run -- docker build  --no-cache \
--secret id=nginx-key,env=NGINX_KEY \
--secret id=nginx-crt,env=NGINX_CRT \
--secret id=agent-crt,env=AGENT_CRT \
--secret id=agent-key,env=AGENT_KEY \
--build-arg NMS_HOST="my_nms_host" \
-t nginx_plus_agent --load .
```

### Run an NGINX Plus with NGINX Agent Container

To run the NGINX Plus container, execute the following command:

```bash
export NMS_SERVER_HOST="my_nms_host_url"
docker run --name nginx-plus-agent -d \
    -e NMS_SERVER_HOST \
    -p 80 \
    -p 443 \
    nginx_plus_agent
```

To run the NGINX Plus container, using 1Password to store the NMS Host information, execute the following command:

```bash
export NMS_SERVER_HOST="op://Work/nms/URL"
op run -- docker run --name nginx-plus-agent -d \
    -e NMS_SERVER_HOST \
    -p 80 \
    -p 443 \
    nginx_plus_agent
```

## Publish Container to GitHub Packages

The steps below highlight how to publish your NGINX Plus container into GitHub Packages so you can use your NGINX Plus container in Kubernetes.

**_NOTE:_** For this step, you will need to [create a GitHub Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).

**_NOTE:_** Please ensure your GitHub package is private; this is the default behavior.

```bash
export GH_PAT="my github personal access token"
docker echo $GH_PAT | login ghcr.io --username my_github_username --password-stdin
docker tag nginxplus ghcr.io/my_github_username/nginxplus:latest
docker push ghcr.io/my_github_username/nginxplus:latest
```

or with 1Password:

```bash
op run -- docker login ghcr.io \
    -u $(op read "op://Work/GitHub Publish NGINX Packages/username") \
    -p $(op read "op://Work/GitHub Publish NGINX Packages/token")
op run -- docker tag nginxplus ghcr.io/$(op read "op://Work/GitHub Publish NGINX Packages/username")/nginxplus:latest
op run -- docker push ghcr.io/$(op read "op://Work/GitHub Publish NGINX Packages/username")/nginxplus:latest
```

**_NOTE:_** Docker will complain that supplying the password via the CLI is insure.  We are not actually entering the password in the CLI so this is a false warning.  However, if you want to avoid this message you can leverage the below login process.

```bash
op run --no-masking -- \
    echo $(op read "op://Work/GitHub Publish NGINX Packages/token") | \
    docker login ghcr.io \
    -u $(op read "op://Work/GitHub Publish NGINX Packages/username") \
    --password-stdin
```
