# NGINX Plus Container

Create an NGINX Plus container and publish it to GitHub Packages

## Build NGINX PLus Container

Download the repository certificate and key from [MyF5](https://my.f5.com) and name them *nginx-repo.crt* and *nginx-repo.key*.

```bash
docker build  --no-cache --secret id=nginx-key,src=nginx-repo.key --secret id=nginx-crt,src=nginx-repo.crt -t nginxplus --load .
```
