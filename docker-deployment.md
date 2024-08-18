## How to deploy a multi container application to a docker server

Instructions below apply to a specific use case where we need to deploy all the application services to a single fat server that is running only docker.

No kubernetes, swarm, or any other container orchestration is required for this.

Instructions work equally good on a local LAN environment as well as on some virtual machine in a VPC in the cloud.

The setup also allows to integrate Jenkins or any other CI/CD tool for continuous deployments and updates. Though not covered here.

Instructions are Ubuntu/Linux specific. If running on any other platform, commands and paths should be adopted accordingly. 

It is assumed that docker is already installed along with compose plugin. If no, then follow the installation instructions here; https://docs.docker.com/engine/install/ubuntu/

### Step 1 - Start a docker registry

Follow these steps to start a local registry for docker container images.

1. Create an empty folder and move into it;
   ```shell
   mkdir ~/registry ; cd ~/registry
   ```
3. Create `docker-compose.yaml` file in folder with these contents;
   ```yaml
   services:
     registry:
       image: registry:latest
       ports:
         - 5000:5000
       restart: unless-stopped
   ```
Step 1.a - Allow docker command to push to the registry

Step 2 - Create a docker compose file that defines application components

Step 3 - Build the application container image

Step 4 - Push the image to the registry

Step 5 - Start/restart the services
