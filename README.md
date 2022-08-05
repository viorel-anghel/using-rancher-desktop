# Rancher desktop for developer laptops

eSolutions July 2022

Rancher Desktop can be used as a replacement for Docker Desktop. It is a **free and open-source** graphical application that provides Docker and Kubernetes for any desktop/laptop with **MacOS (both on Intel and Apple Silicon), Windows, and Linux**. Full documentation is at https://docs.rancherdesktop.io/

## Clean your laptop before

If you already have Docker Desktop, you will need to stop it and maybe later uninstall it.

On Linux or Macos, if you have `kind` and / or `docker` installed it would be a good ideea to cleanup everything before installing Rancher Desktop. If you cannot afford this, you will need to at least stop the docker engine, probably with a command like `sudo systemctl stop docker`, eventually prevent it to start on boot with `sudo systemctl disable docker`.

To cleanup a `kind + docker` Linux setup, try something along those lines:
```
kind delete cluster
sudo docker ps # list all running containers
sudo docker rm -f <ID1> <ID2>... etc  # stop and delete containers with <ID>
sudo docker system prune --volumes
dpkg -l | grep docker   # list all installed packaged show only docker
sudo apt purge docker-ce ... # uninstall all docker packages
```

## Install Rancher desktop and options

Follow steps from https://docs.rancherdesktop.io/getting-started/installation

For Ubuntu Linux, you will need to `sudo apt install pass` and to generate a gpg key as explained in there. Then continue with "Installation via .deb Package". 

Then start Rancher desktop, search it in your graphical menu system.

On first start, you will be presented with some choices. Most of the defaults are ok, except at "Container runtime" where we recommend selecting the option **"dockerd (moby)"** instead of "containerd". This way you will be able to use familliar docker commands.

![preferences](screenshot2.png?raw=true "preferences")

Wait until the progress bar in the left-down side is finishing. Then **start a NEW shell** and try

```
docker ps
kubectl get nodes
```

If those works you're all set. You may close the Rancher desktop main window and if needed access it from the small icon in the taskbar:

![taskbar icon](screenshot1.png?raw=true "taskbar icon")


## Not working on newer Apple macbooks - PATH problems

If the above commands fails, you may have a PATH problem. The Rancher desktop will add something like this into your `~/.bashrc` file:

```
### MANAGED BY RANCHER DESKTOP START (DO NOT EDIT)
export PATH="/Users/vang/.rd/bin:$PATH"
### MANAGED BY RANCHER DESKTOP END (DO NOT EDIT)
```

because it installs all it's binaries in that `.rd/bin` directory. But on newer Macs, `zsh` is the default shell instead of `bash` so you will need to copy those lines from `~/.bashrc` in a file called `~/.zshrc`.

## Using Rancher desktop

### Building images and using them in kubernetes

If you just want to build an image and use it in your local Kubernetes, you don't need an image repository. For example, i'm building this image and imediately start a pod from it:

```
docker build -t dummy:0.7 .
docker images
kubectl delete pod dummy              # delete dummy pod if exists
kubectl run dummy --image=dummy:0.7   # start a new dummy pod from this image
kubectl get pods
```

### Building images on M1 Macbooks

As shown above, you can `docker build` images and run them on your laptop. But if you try to push them to Docker Hub you will notice, on Docker hub webinterface, the image is built for the arm64 architecture. You will frequently have Kubernetes clusters with amd64 / x86 architecture so you need the docker image to be on that arch. Fortunately, there is a way to build images multi-architecture. Read more on https://blog.jaimyn.dev/how-to-build-multi-architecture-docker-images-on-an-m1-mac/

This is an example on how I do it for my dummy image:
```
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t vvang/dummy:0.8 .
docker push vvang/dummy:0.8
#docker ps | grep buildx # remove that container if you wish
```

If you look now at the Docker hub webinterface you will see the new image is multi-arch:

![docker hub](docker-hub.png?raw=true "docker hub")

Docker and Kubernetes will auto-select the right architecture when pulling the image.

### Compiling Go programs on M1 Macbooks

When you are compiling GO programs, by default the compiler will target your processor and OS. For example, `go build main.go` will produce a binary which you can run on your M1 Macbook.

If you try to run this on another architecture you will get an `exec format error`.

For the go language, you will need to compile with something like `GOOS=linux GOARCH=amd64 go build main.go`. Those two environment variables will tell go compiler which is the target architecture and operating system.

### Nginx vs. Traefik ingress controller

Rancher desktop uses Traefik as the default ingress controller in this local Kubernetes. On our prod/dev clusters we use Nginx. 

If you want to be as close as production to that, you may switch to Nginx by using this info https://docs.rancherdesktop.io/how-to-guides/setup-NGINX-Ingress-Controller . At step 1  you will need to exit and restart Rancher desktop.


