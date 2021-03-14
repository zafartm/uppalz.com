# Run leiningen using docker

## leind
Save the following command in a file named `leind` in project's root directory.
```sh
docker run -it --rm -u 1000:1000 -v ~/.m2:/tmp/.m2 -v $(pwd):/app -w /app -e HOME=/tmp -e LEIN_JVM_OPTS="-Duser.home=/tmp" clojure:openjdk-8-lein lein $@
```
Then make the file executable: `chmod +x leind`


## Usage examples

1. `./leind -v`
1. `./leind -h`
1. `./leind new app myapp --to-dir .`
1. `./leind repl`
1. `./leind run`

## Explanation

### `docker run -it`
Runs the docker container in interactive mode (tty and input attached).
### `--rm`
Removes the container as soon as it stops.
### `-u 1000:1000`
User id and group id to be used. If skipped, it uses `root:root` by default.
### `-v ~/.m2:/tmp/.m2`
Mounts local `~/.m2` directory within docker container at `/tmp/.m2`. This is needed to avoid re-downloading of maven artifacts on each run. Remember leiningen uses maven repositories for its artifacts.
### `-v $(pwd):/app`
Mounts local working directory within docker container at `/app`. This is needed so that local files are available to leiningen that is running within a container.
### `-w /app`
Sets working directory within the container.
### `-e HOME=/tmp`
Sets $HOME environemnt variable within the container to point to `/tmp` directory
### `LEIN_JVM_OPTS="-Duser.home=/tmp"`
Tells JVM that `/tmp` is the user.home.
### `clojure:openjdk-8-lein`
Docker container image that is being run. `clojure` is the image name, `openjdk-8-lein`is the version. See [maven page at docker hub](https://hub.docker.com/_/clojure) for all available versions.
### `lein`
This is the actual command to start leiningen process.
### `$@`
This takes commandline arguments from the `leind` script and passes on to `lein` command.
