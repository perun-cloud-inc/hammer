# perun_hammer

This CLI tool main goal is to enable developers to debug their containerized app as if they are debugging it localy inside their favorite IDE (currently only vscode is supported).
the containerized app will run in a local docker container while communicating to the external services whether running localy or inside a k8s cluster.

## Some definitions first ##

**Perun service** - a cotainerized application.
we support multiple types of services :
- The docker service : its a service representing a docker image where the command and arg can be overriten if need be.
- The local service : a representation of a local folder where the application repository is found, a Dockefile should already be created at the root of this location, defining the docker container of the application.
  *currently node and python applications are only supported this should be specified in the yaml as well.*
 
*Put the service schema here*

**Perun environment** - a grouping of services that make the whole application

**Perun workspace** - a logical grouping of environemnts

## Environment definition ##
The first step of using perun hammer is to define the application in a yaml file, the file will contain the various services of the application and their dependencies.
To make things easier for k8s cluster users, we allow environment import which will connect to the specified k8s namespace and import the environment running there as a perun environment yaml.

each service in the environemnt yaml can define its dependencies influencing the order of the services load when working in local mode.
In addition, each service can define its environment variables which will be injected into the running local process when loaded.


## Pre-requisites

* A running Docker setup on the local machine
* Bridge to kubenetese extension in vscode https://marketplace.visualstudio.com/items?itemName=mindaro.mindaro *hybrid mode requirement

## The Hammer CLI
```
Usage:
  hammer [command]

Available Commands:
  activate    activate Perun environemnt in a target workspace
  apply       apply the provided env on a workspace, in dry run mode the environment will be analyzed and persisted but not loaded into the target deployment
  completion  Generate the autocompletion script for the specified shell
  deactivate  deactivate Perun environemnt in a target workspace
  deploy      A brief description of your command
  destroy     Destroys and clears given workspace
  generate    generate debug config for supplied service
  help        Help about any command
  import      import target environment into a workspace
  init        initialize empty Perun workspace
  list        list existing perun workspaces
  login       login to Perun platform: hammer login Username Password
  nomad       A brief description of your command
  sg          Adds ip to the SG
  synchronize synchronize Perun environemnt in a target workspace

Flags:
      --config string   config file (default is $HOME/.hammer.yaml)
  -h, --help            help for hammer
  -t, --toggle          Help message for toggle

Use "hammer [command] --help" for more information about a command.
```

## Basic steps for debugging a service
### Local Mode ###
1. Creating the perun environemnt yaml according to the following schema
2. Applying the desired perun environment localy... this will localy load all the docker containers in that environemnt into the targeted perun workspace.
```
hammer init -w <workspace-name>
hammer apply -w <workspace-name> -e <env-name> -p <path-to-the-env-yml>
```
3. generating the debug configuration of the desired service, this will generate vscode launch configuration and persist it under the given repository location of said service.
```
hammer generate -w <workspace-name> -e <env-name> -s <service-name-to-debug>
```
4. open vscode to the service workspace. you will be able now to run the debug target in the generated launch configuration and start your debugging session. this will effectively pause the original service docker container ad load your debug container while routing all traffic to it.
5. once debug session is over all the above re-routing will be reverted and the original container will be back in a running state.

### Hybrid Mode ###
1. import a k8s namespace to a local perun environment representation
```
hammer import -t k8s -w <target-workspace-name> -n <namespace-to-import> -c <k8s-cluster-name>
```
2. tweak the generated perun environemnt yaml, specificaly the service you want to debug by pointing to the local repository location where the source code is found for that service and set the repo location. define as well the run command and arguments whith which you start your service.
3. generating the debug configuration of the desired service that you want to debug, this will generate vscode launch configuration and persist it under the given repository location of said service.
```
hammer generate -w <workspace-name> -e <env-name> -s <service-name-to-debug>
```
4. open vscode to the service workspace. you will be able now to run the debug target in the generated launch configuration and start your debugging session. this will effectively put aside the original k8a pod and load your local debug environment while routing all traffic to it and from it, as well as modifying the local host file so that the debugged service will be able to reach all existing services in the k8s cluster.
*please make sure that the current namespace is the one you are connecting to
5. once debug session is over all the above re-routing and modifications will be reverted and the original pod will be back in a running order.


