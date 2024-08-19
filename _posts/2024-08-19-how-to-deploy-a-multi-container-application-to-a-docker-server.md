---
layout: post
title:  "How to deploy a multi container application to a docker server"
---

# How to deploy a multi container application to a docker server

Instructions below apply to a specific use case where we need to deploy all the application services to a single fat server that is running only docker.

No kubernetes, swarm, or any other container orchestration is required for this.

Instructions work equally good on a local LAN environment as well as on some virtual 
machine in a VPC in the cloud.

The setup also allows to integrate Jenkins or any other CI/CD tool for continuous 
deployments and updates. Though not covered here.

Instructions are Ubuntu/Linux specific. If running on any other platform, commands 
and paths should be adopted accordingly.

Some of the steps below are to be performed on **the target server** and the rest on **the build server**.

Target server needs to have a known IP address that is reachable from the build server. Here 
the ip is assumed to be **192.168.56.110**

Build server can be a developer's local machine. It can be a Jenkins server as well. As long as 
it can execute `docker build` and `docker push` commands, it is good enough for the example in this article.

It is assumed that the docker service is already installed along with compose plugin.
We need it on both the target server and well as the build server.
If no, then follow the installation instructions here; https://docs.docker.com/engine/install/ubuntu/

### Step 1 - On the target server - Start a docker registry

We need to start a registry on the target server where application needs to be deployed.
This is required so that pre-built application images can be uploaded to the server. The
process is very simple.

Follow these steps to start a local registry for docker container images.

1. Create an empty folder and move into it;
   ```shell
   mkdir ~/registry
   cd ~/registry
   ```
1. Create `docker-compose.yaml` file in the folder with these contents;
   ```yaml
   services:
     registry:
       image: registry:latest
       ports:
         - 50005:5000
       restart: unless-stopped
   ```
1. Start the registry;
   ```shell
   docker compose up -d
   ```
This starts a registry that is listening on port `50005` for `push` and `pull` requests. If 
this port needs to be changed, alter the `ports` section in the `docker-compose.yaml` file.

##### WARNING Note !!!
The registry started above works on plain HTTP and requires no authentication. So make sure 
that the port is secured in the firewall. Allow traffic to it only from trusted hosts.  


### Step 2 - On the build server - Allow docker command to push to the registry
This step is to be performed on the build server. It could be the Jenkins machine or the 
developer's own desktop.

Once registry is ready, we need to configure the docker service, so that it can push images 
to the registry.

With default settings, `docker push` command refuses to push images over insecure plain http. 
We need to add the IP address and port number of the registry server in `/etc/docker/daemon.json` 
file, so that it recognises the registry and allows pushes to it.

1. Edit or create the file `/etc/docker/daemon.json` and make sure that `insecure-registries` 
   property exists and has an entry that points to the IP address and port number of the host 
   where registry service is running;
   ```json
   {
      "insecure-registries" : ["192.168.56.110:50005"] 
   }
   ```
2. Restart docker service so that it reloads the config change;
   ```shell
   sudo systemctl restart docker
   ```


### Step 3 - On the target server - Create a docker compose file that defines application components

Once registry and docker service setup is completed, we can come to the actual application services.
We need another docker-compose.yaml file to define these;

1. Create another empty folder and move to it;
   ```shell
   mkdir ~/app
   cd ~/app
   ```
2. Create a `docker-compose.yaml` file. Here we have an example that starts;
   1. Mongo db server
   2. Redis server
   3. Api server (backend)
   4. UI server (frontend)
   
   Example contents;
   ```yaml
   volumes:
     mongodb-data:

   services:
      mongodb-server:
        image: mongo:7
        volumes:
          - mongodb-data:/data/db
        ports:
          - 27017:27017
        restart: unless-stopped

      redis-server:
        image: redis:7-alpine
        ports:
          - 6379:6379
        restart: unless-stopped
         
      backend-server:
        image: 127.0.0.1:50005/app-backend:latest
        pull_policy: always
        ports:
          - 8001:8000
        environment:
          MONGO_HOST: mongodb-server
          REDIS_HOST: redis-server
        env_file:
          - backend.env
        restart: unless-stopped
   
      frontend-server:
        image: 127.0.0.1:50005/app-frontend:latest
        pull_policy: always
        ports:
          - 8002:8000
        environment:
          BACKEND_HOST: backend-server
        env_file:
          - frontend.env
        restart: unless-stopped
   ```
   This is just an example content that needs to be adjusted as per needs.
   Noteable points in this example;
   1. `mongodb-server` and `redis-server` are standard service blocks. Nothing special is needed here.
   2. Note the `image` property in `backend-server` and `frontend-server` blocks. This points to the
      loopback address with the port number of the registry service started in step 1 above.
      
      This essentially is telling the docker to get the `app-backend` and `app-frontend` images from the 
      registry running locally.
      
      As no image is pushed to the registry yet, any attempt to start services using this docker compose 
      file shall fail.
   3. Also note the `pull_policy: always` property in both backend and frontend service definitions. This 
      seems redundant the first time as docker shall pull the image anyway. But the option becomes useful
      for subsequent runs, specially after some code changes. As it shall force the docker service to always 
      pull the latest image from the registry. 

   4. The `env_file` property in both frontend and backend server blocks is referring to 
      local config files that define application's runtime environment.
    
      Either create both these `backend.env` and `frontend.env` files in the ~/app folder. Or remove the 
      `env_file` property from docker compose file if no custom env is needed.     

### Step 4 - On the build server - Build and Push the application container images

While building the docker container image, only requirement is to tag it correctly. The tag needs to have 
three required parts;
- Hostname or IP address of the docker registry. Here in this example it needs to be `192.168.56.110`
- Port number of the docker registry. Here it needs to be `50005`
- Image name. Here it need to be `app-backend` for the backend service and `app-frontend` for the frontend
  service.
1. Command to build and tag backend image; Run this where backend source code is present along with the `Dockerfile`
   ```shell
   docker build -t 192.168.56.110:50005/app-backend .
   ```
1. Command to build and tag frontend image; Run this where frontend source code is present along with the `Dockerfile`
   ```shell
   docker build -t 192.168.56.110:50005/app-frontend .
   ```
1. Commands to push both images to the registry;
   ```shell
   docker push 192.168.56.110:50005/app-backend
   docker push 192.168.56.110:50005/app-frontend
   ```



### Step 5 - On the build server - Start/restart the services

Once images are pushed successfully to registry running on the target server, we can start the 
application services that were defined in the step 3 above using this command;

```shell
cd ~/app
docker compose up -d
```

Note that nothing special is needed at this step as standard `docker compose up` command will do the needed.
Due to the `pull_policy` property in the docker-compose.yaml file in step 3, every time we issue this command, 
it goes to the registry first to see if a new image is available to download. If yes, it pulls it and restarts
the related services.   


### The code-build-deploy-test cycle after the initial setup

When on the developer's machine;
1. Make the required code changes. 
2. Build, tag and push the image using `docker` commands as described in step 4 above
3. Issue `docker compose up -d` on the target server

When Jenkins or some other build server is involved;
1. Make the required code changes and push to git repository.
2. Run the Jenkins job.
   The job shall check out the code from the git repository then run a shell script like this as a build step;
   ```shell
   docker build -t 192.168.56.110:50005/app-backend .
   docker push 192.168.56.110:50005/app-backend
   ssh -i identity-file.pem user@192.168.56.110 "cd ~/app ; docker compose up -d"
   ```
