# Run maven using docker

## mvnd
Save the following command in a file named `mvnd` in project's root directory.
```sh
docker run -it --rm -u 1000:1000 -v ~/.m2:/maven/.m2 -v $(pwd):/app -w /app -p 8080:8080 -e MAVEN_CONFIG=/maven/.m2 maven:3-openjdk-8 mvn -Duser.home=/maven $@
```
Then make the file executable: `chmod +x mvnd`

## Usage examples

1. `./mvnd spring-boot run`
1. `./mvnd compile`
1. `./mvnd clean package`
1. `./mvnd archetype:generate`

## Explanation

### `docker run -it`
Runs the docker container in interactive mode (tty and input attached).
### `--rm`
Removes the container once done.
### `-u 1000:1000`
User id and group id to be used. If skipped, it uses `root:root` by default.
### `-v ~/.m2:/maven/.m2`
Mounts local `~/.m2` directory within docker container at `/maven/.m2`. This is needed to avoid re-downloading of maven artifacts on each run.
### `-v $(pwd):/app`
Mounts local working directorty within docker container at `/app`. This is needed so that local files are available to maven that is running within a container.
### `-w /app`
Sets working directory within the container.
### `-p 8080:8080`
Maps port 8080 to accept network connections. This is needed when doing something like `spring-boot:run` or `tomcat:run`.
If no service is started, this can be omitted.
If service is running on a different port, adjust it accordingly.
If multiple services (e.g. Tomcat and LiveReload in a spring-boot MVC app) are started, then repeat the option multiple times for each port.
### `-e MAVEN_CONFIG=/maven/.m2`
This is required when `-u 1000:1000` option is used. Otherwise skip it.
### `maven:3-openjdk-8`
Docker container image that is being run. `maven` is the image name, `3-openjdk-8`is the version. See [maven page at docker hub](https://hub.docker.com/_/maven) for all available versions.
### `mvn`
This is the actual command to start maven. Remaining options are argumnets for this command.
### `-Duser.home=/maven`
This is required when `-u 1000:1000` option is used. Skip otherwise.
### `$@`
This takes commandline arguments of the `mvnd` script and passes on to `mvn` command.
