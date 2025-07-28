# What is a devfile

You can use devfiles to automate and simplify your development process
by adopting the existing devfiles that are available in the [public community registry](https://registry.devfile.io/viewer)
or by authoring your own devfiles to record custom instructions to
configure and run your build environment as a YAML-formatted text file.
You can make these devfiles available in the supporting build tools and
IDEs that can automatically process the devfile instructions to configure
and build a running application from a development project.

Using the recommended best practices from the devfile, the tools and IDE
can:

- Take in the repository hosting your application source code.

- Build your code.

- Run your application on your local container.

- Deploy your application to cloud-native containers.

# Benefits of devfile

## Why devfiles?

With devfiles, you can make workspaces composed of multiple containers.
In these containers, you can create any number of identical workspaces
from the same devfile. If you create multiple workspaces, you can share
your devfile with different teams. By sharing a single devfile across
multiple teams, you can ensure that each team has the same user
experience and build, run, deploy behaviors.

Devfiles include the following features:

- Guidance for using runtime images

- Example code

- Build and CI commands

- Deployment options

Devfiles have the following benefits:

- Reduce the gap between development and deployment

- Find available devfile stacks or samples in a devfile registry

- Produce consistent build and run behaviors

## Who is devfile for?

- [Application developers](./application-developer)
- [Enterprise architects and runtime providers](./enterprise-architect-and-runtime-provider)
- [Registry administrators](./registry-administrator)
- [Technology and tool builders](./technology-and-tool-builders)

# Devfile ecosystem

## Create, Share and Consume Devfiles

Organizations looking to standardize their development environment can do so by adopting devfiles. In the simplest case, developers can just consume the devfiles that are available from the public community registry. If your organization needs custom devfiles that are authored and shared internally, then you need a role based approach so developers, devfile authors, and registry administrators can interact together.

### Create

A devfile author, also known as a runtime provider, can be an individual or a group representing a runtime vendor. Devfile authors need sound knowledge of the supported runtime so they can create devfiles to build and run applications.

If a runtime stack is not available in the public registry, an organization can choose to develop their own and keep it private for their in-house development.

### Share

The public community registry is managed by the community and hosted by Red Hat. Share your devfile to the public community registry so other teams can benefit from your application.

If an organization wants to keep their own devfiles private but wishes to share with other departments, they can assign a registry administrator. The registry administrator deploys, maintains, and manages the contents of their private registry and the default list of devfile registries available in a given cluster.

### Consume

Developers can use the supported tools to access devfiles. Many of the existing tools offer a way to register or catalog public and private devfile registries which then allows the tool to expose the devfiles for development.

In addition, each registry comes packaged with an index server and a registry viewer so developers can browse and view the devfile contents before deciding which ones they want to adopt.

Developers can also extend an existing parent devfile to customize the workflow of their specific application. The devfile can be packaged as part of the application source to ensure consistent behavior when moving across different tools.

> **Note:** Tools that support the devfile spec might have varying levels of support. Check their product pages for more information.

# Devfile validation rules

## Id and name:
`^[a-z0-9]([-a-z0-9]*[a-z0-9])?$`

The restriction is added to allow easy translation to K8s resource names, and also to have consistent rules for both `name` and `id` fields.

The validation will be done as part of schema validation, the rule will be introduced as a regex in schema definition, any objection of the rule in devfile will result in a failure.

- Limit to lowercase characters i.e., no uppercase allowed
- Limit within 63 characters
- No special characters allowed except dash(-)
- Start with an alphanumeric character
- End with an alphanumeric character

## Endpoints:
- All the endpoint names are unique across components
- Endpoint ports must be unique across `container` components -- two `container` components cannot have the same `targetPort`, but one `container` component may have two endpoints with the same `targetPort`. This restriction does not apply to `container` components with `dedicatedPod` set to `true`.


## Commands:
1. `id` must be unique
2. `composite` command:
    - Should not reference itself via a subcommand
    - Should not indirectly reference itself via a subcommand which is a `composite` command
    - Should reference a valid devfile command
3. `exec` command should: map to a valid `container` component
4. `apply` command should: map to a valid `container`/`kubernetes`/`openshift`/`image` component
5. `{build, run, test, debug, deploy}`, each kind of group can only have one default command associated with it. If there are multiple commands of the same kind without a default, a warning will be displayed.

## Components:
Common rules for all components types:
- `name` must be unique

### Container component 
1. The container components must reference a valid volume component if it uses volume mounts, and the volume components are unique
2. `PROJECT_SOURCE` or `PROJECTS_ROOT` are reserved environment variables defined under env, cannot be defined again in `env`
3. The annotations should not have conflict values for same key, except deployment annotations and service annotations set for a container with `dedicatedPod=true`
4. Resource requirements, e.g. `cpuLimit`, `cpuRequest`, `memoryLimit`, `memoryRequest`, must be in valid quantity format; and the resource requested must be less than the resource limit (if specified).

### Plugin component
- Commands in plugins components share the same commands validation rules as listed above. Validation occurs after overriding and merging, in flattened devfile
- Registry URL needs to be in valid format

### Kubernetes & OpenShift component 
- `uri` needs to be a valid URI format

### Image component 
- An `image` component's git source cannot have more than one remote defined. If checkout remote is mentioned, validate it against the remote configured map

## Events:
1. `preStart` and `postStop` events can only be `apply` commands
2. `postStart` and `preStop` events can only be `exec` commands
3. If `preStart` and `postStop` events refer to a `composite` command, then all containing commands need to be `apply` commands.
4. If `postStart` and `preStop` events refer to a `composite` command, then all containing commands need to be `exec` commands.

## Parent:
- Share the same validation rules as listed above. Validation occurs after overriding and merging, in flattened devfile

## Starter projects:
- Starter project entries cannot have more than one remote defined
- If `checkoutFrom.remote` is mentioned, validate it against the starter project remote configured map

## Projects
- `checkoutFrom.remote` is mandatory if more than one remote is configured
- If checkout remote is mentioned, validate it against the starter project remote configured map

# Devfile Registry

You can browse for the ready-to-use examplesof devfiles for various sample projects and stacks to automate and simplify your development process
available in the [public community registry](https://registry.devfile.io/viewer)

# Devfile schema - Version 2.3.0

The Devfile JSON Schema v.2.3.0 can be downloaded from https://devfile.io/devfile-schemas/2.3.0.json

Devfile describes the structure of a cloud-native devworkspace and development environment.

**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#attributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|[**commands**](#commands)|`object[]`|Predefined, ready-to-use, devworkspace-related commands |no|
|[**components**](#components)|`object[]`|List of the devworkspace components, such as editor and plugins, user-provided containers, or other types of components |no|
|[**dependentProjects**](#dependentprojects)|`object[]`|Additional projects related to the main project in the devfile, contianing names and sources locations |no|
|[**events**](#events)|`object`|Bindings of commands to events. Each command is referred-to by its name. |no|
|[**metadata**](#metadata)|`object`|Optional metadata |no|
|[**parent**](#parent)|`object`|Parent devworkspace template |no|
|[**projects**](#projects)|`object[]`|Projects worked on in the devworkspace, containing names and sources locations |no|
|**schemaVersion**|`string`|Devfile schema version Pattern: `^([2-9])\.([0-9]+)\.([0-9]+)(\-[0-9a-z-]+(\.[0-9a-z-]+)*)?(\+[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?$` |yes|
|[**starterProjects**](#starterprojects)|`object[]`|StarterProjects is a project that can be used as a starting point when bootstrapping new projects |no|
|[**variables**](#variables)|`object`|Map of key-value variables used for string replacement in the devfile. Values can be referenced via {{variable-key}} to replace the corresponding value in string fields in the devfile. Replacement cannot be used for |no|

**Additional Properties:** not allowed **Example**

```yaml
schemaVersion: 2.3.0
```


## attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
## commands\[\]: array

Predefined, ready-to-use, devworkspace-related commands


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**apply**](#commandsapply)|`object`|Command that consists in applying a given component definition, typically bound to a devworkspace event. |yes|
|[**attributes**](#commandsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|[**composite**](#commandscomposite)|`object`|Composite command that allows executing several sub-commands either sequentially or concurrently |no|
|[**exec**](#commandsexec)|`object`|CLI Command executed in an existing component container |yes|
|**id**|`string`|Mandatory identifier that allows referencing this command in composite commands, from a parent, or in events. Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * exec

  **Option 2 (alternative):** 
**Required Properties:**

  * apply

  **Option 3 (alternative):** 
**Required Properties:**

  * composite

**Example**

```yaml
- apply:
    group: {}
  attributes: {}
  composite:
    group: {}
  exec:
    env:
      - {}
    group: {}

```


### commands\[\]\.apply: object

Command that consists in applying a given component definition, typically bound to a devworkspace event.

For example, when an `apply` command is bound to a `preStart` event, and references a `container` component, it will start the container as a K8S initContainer in the devworkspace POD, unless the component has its `dedicatedPod` field set to `true`.

When no `apply` command exist for a given component, it is assumed the component will be applied at devworkspace start by default, unless `deployByDefault` for that component is set to false.


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**component**|`string`|Describes component that will be applied |yes|
|[**group**](#commandsapplygroup)|`object`|Defines the group this command is part of |yes|
|**label**|`string`|Optional label that provides a label for this command to be used in Editor UI menus for example |no|

**Additional Properties:** not allowed **Example**

```yaml
group: {}

```


#### commands\[\]\.apply\.group: object

Defines the group this command is part of


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**isDefault**|`boolean`|Identifies the default command for a given group kind |no|
|**kind**|`string`|Kind of group the command is part of Enum: `"build"`, `"run"`, `"test"`, `"debug"`, `"deploy"` |yes|

**Additional Properties:** not allowed 
### commands\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
### commands\[\]\.composite: object

Composite command that allows executing several sub-commands either sequentially or concurrently


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**commands**](#commandscompositecommands)|`string[]`|The commands that comprise this composite command ||
|[**group**](#commandscompositegroup)|`object`|Defines the group this command is part of |yes|
|**label**|`string`|Optional label that provides a label for this command to be used in Editor UI menus for example ||
|**parallel**|`boolean`|Indicates if the sub-commands should be executed concurrently ||

**Additional Properties:** not allowed **Example**

```yaml
group: {}

```


#### commands\[\]\.composite\.commands\[\]: array

The commands that comprise this composite command


**Items**

**Item Type:** `string` 
#### commands\[\]\.composite\.group: object

Defines the group this command is part of


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**isDefault**|`boolean`|Identifies the default command for a given group kind |no|
|**kind**|`string`|Kind of group the command is part of Enum: `"build"`, `"run"`, `"test"`, `"debug"`, `"deploy"` |yes|

**Additional Properties:** not allowed 
### commands\[\]\.exec: object

CLI Command executed in an existing component container


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**commandLine**|`string`|The actual command-line string
Special variables that can be used:
- `$PROJECTS_ROOT`: A path where projects sources are mounted as defined by container component's sourceMapping.
- `$PROJECT_SOURCE`: A path to a project source ($PROJECTS_ROOT/\<project-name\>). If there are multiple projects, this will point to the directory of the first one. |yes|
|**component**|`string`|Describes component to which given action relates |yes|
|[**env**](#commandsexecenv)|`object[]`|Optional list of environment variables that have to be set before running the command |no|
|[**group**](#commandsexecgroup)|`object`|Defines the group this command is part of |yes|
|**hotReloadCapable**|`boolean`|Specify whether the command is restarted or not when the source code changes. If set to `true` the command won't be restarted. A *hotReloadCapable* `run` or `debug` command is expected to handle file changes on its own and won't be restarted. A *hotReloadCapable* `build` command is expected to be executed only once and won't be executed again. This field is taken into account only for commands `build`, `run` and `debug` with `isDefault` set to `true`.
Default value is `false` |no|
|**label**|`string`|Optional label that provides a label for this command to be used in Editor UI menus for example |no|
|**workingDir**|`string`|Working directory where the command should be executed
Special variables that can be used:
- `$PROJECTS_ROOT`: A path where projects sources are mounted as defined by container component's sourceMapping.
- `$PROJECT_SOURCE`: A path to a project source ($PROJECTS_ROOT/\<project-name\>). If there are multiple projects, this will point to the directory of the first one. |no|

**Additional Properties:** not allowed **Example**

```yaml
env:
  - {}
group: {}

```


#### commands\[\]\.exec\.env\[\]: array

Optional list of environment variables that have to be set before running the command


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**name**|`string`||yes|
|**value**|`string`||yes|

**Item Additional Properties:** not allowed **Example**

```yaml
- {}

```


#### commands\[\]\.exec\.group: object

Defines the group this command is part of


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**isDefault**|`boolean`|Identifies the default command for a given group kind |no|
|**kind**|`string`|Kind of group the command is part of Enum: `"build"`, `"run"`, `"test"`, `"debug"`, `"deploy"` |yes|

**Additional Properties:** not allowed 
## components\[\]: array

List of the devworkspace components, such as editor and plugins, user-provided containers, or other types of components


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#componentsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|[**container**](#componentscontainer)|`object`|Allows adding and configuring devworkspace-related containers |yes|
|[**image**](#componentsimage)|`object`|Allows specifying the definition of an image for outer loop builds |yes|
|[**kubernetes**](#componentskubernetes)|`object`|Allows importing into the devworkspace the Kubernetes resources defined in a given manifest. For example this allows reusing the Kubernetes definitions used to deploy some runtime components in production. |no|
|**name**|`string`|Mandatory name that allows referencing the component from other elements (such as commands) or from an external devfile that may reference this component through a parent or a plugin. Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|[**openshift**](#componentsopenshift)|`object`|Allows importing into the devworkspace the OpenShift resources defined in a given manifest. For example this allows reusing the OpenShift definitions used to deploy some runtime components in production. |no|
|[**volume**](#componentsvolume)|`object`|Allows specifying the definition of a volume shared by several other components |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * container

  **Option 2 (alternative):** 
**Required Properties:**

  * kubernetes

  **Option 3 (alternative):** 
**Required Properties:**

  * openshift

  **Option 4 (alternative):** 
**Required Properties:**

  * volume

  **Option 5 (alternative):** 
**Required Properties:**

  * image

**Example**

```yaml
- attributes: {}
  container:
    annotation:
      deployment: {}
      service: {}
    endpoints:
      - annotation: {}
        attributes: {}
        exposure: public
        protocol: http
    env:
      - {}
    sourceMapping: /projects
    volumeMounts:
      - {}
  image:
    dockerfile:
      devfileRegistry: {}
      git:
        checkoutFrom: {}
        remotes: {}
  kubernetes:
    endpoints:
      - annotation: {}
        attributes: {}
        exposure: public
        protocol: http
  openshift:
    endpoints:
      - annotation: {}
        attributes: {}
        exposure: public
        protocol: http
  volume: {}

```


### components\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
### components\[\]\.container: object

Allows adding and configuring devworkspace-related containers


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#componentscontainerannotation)|`object`|Annotations that should be added to specific resources for this container |no|
|[**args**](#componentscontainerargs)|`string[]`|The arguments to supply to the command running the dockerimage component. The arguments are supplied either to the default command provided in the image or to the overridden command. |no|
|[**command**](#componentscontainercommand)|`string[]`|The command to run in the dockerimage component instead of the default one provided in the image. |no|
|**cpuLimit**|`string`||no|
|**cpuRequest**|`string`||no|
|**dedicatedPod**|`boolean`|Specify if a container should run in its own separated pod, instead of running as part of the main development environment pod.
Default value is `false` |no|
|[**endpoints**](#componentscontainerendpoints)|`object[]`||no|
|[**env**](#componentscontainerenv)|`object[]`|Environment variables used in this container. |no|
|**image**|`string`||yes|
|**memoryLimit**|`string`||no|
|**memoryRequest**|`string`||no|
|**mountSources**|`boolean`|Toggles whether or not the project source code should be mounted in the component.
Defaults to true for all component types except plugins and components that set `dedicatedPod` to true. |no|
|**sourceMapping**|`string`|Optional specification of the path in the container where project sources should be transferred/mounted when `mountSources` is `true`. When omitted, the default value of /projects is used. Default: `"/projects"` |no|
|[**volumeMounts**](#componentscontainervolumemounts)|`object[]`|List of volumes mounts that should be mounted is this container. |no|

**Additional Properties:** not allowed **Example**

```yaml
annotation:
  deployment: {}
  service: {}
endpoints:
  - annotation: {}
    attributes: {}
    exposure: public
    protocol: http
env:
  - {}
sourceMapping: /projects
volumeMounts:
  - {}

```


#### components\[\]\.container\.annotation: object

Annotations that should be added to specific resources for this container


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**deployment**](#componentscontainerannotationdeployment)|`object`|Annotations to be added to deployment ||
|[**service**](#componentscontainerannotationservice)|`object`|Annotations to be added to service ||

**Additional Properties:** not allowed **Example**

```yaml
deployment: {}
service: {}

```


##### components\[\]\.container\.annotation\.deployment: object

Annotations to be added to deployment


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


##### components\[\]\.container\.annotation\.service: object

Annotations to be added to service


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


#### components\[\]\.container\.args\[\]: array

The arguments to supply to the command running the dockerimage component. The arguments are supplied either to the default command provided in the image or to the overridden command.

Defaults to an empty array, meaning use whatever is defined in the image.


**Items**

**Item Type:** `string` 
#### components\[\]\.container\.command\[\]: array

The command to run in the dockerimage component instead of the default one provided in the image.

Defaults to an empty array, meaning use whatever is defined in the image.


**Items**

**Item Type:** `string` 
#### components\[\]\.container\.endpoints\[\]: array

**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#componentscontainerendpointsannotation)|`object`|Annotations to be added to Kubernetes Ingress or Openshift Route |no|
|[**attributes**](#componentscontainerendpointsattributes)|`object`|Map of implementation-dependant string-based free-form attributes. |no|
|**exposure**|`string`|Describes how the endpoint should be exposed on the network. - `public` means that the endpoint will be exposed on the public network, typically through a K8S ingress or an OpenShift route. - `internal` means that the endpoint will be exposed internally outside of the main devworkspace POD, typically by K8S services, to be consumed by other elements running on the same cloud internal network. - `none` means that the endpoint will not be exposed and will only be accessible inside the main devworkspace POD, on a local address.
Default value is `public` Default: `"public"` Enum: `"public"`, `"internal"`, `"none"` |no|
|**name**|`string`|Maximal Length: `15` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|Path of the endpoint URL |no|
|**protocol**|`string`|Describes the application and transport protocols of the traffic that will go through this endpoint. - `http`: Endpoint will have `http` traffic, typically on a TCP connection. It will be automaticaly promoted to `https` when the `secure` field is set to `true`. - `https`: Endpoint will have `https` traffic, typically on a TCP connection. - `ws`: Endpoint will have `ws` traffic, typically on a TCP connection. It will be automaticaly promoted to `wss` when the `secure` field is set to `true`. - `wss`: Endpoint will have `wss` traffic, typically on a TCP connection. - `tcp`: Endpoint will have traffic on a TCP connection, without specifying an application protocol. - `udp`: Endpoint will have traffic on an UDP connection, without specifying an application protocol.
Default value is `http` Default: `"http"` Enum: `"http"`, `"https"`, `"ws"`, `"wss"`, `"tcp"`, `"udp"` |no|
|**secure**|`boolean`|Describes whether the endpoint should be secured and protected by some authentication process. This requires a protocol of `https` or `wss`. |no|
|**targetPort**|`integer`|Port number to be used within the container component. The same port cannot be used by two different container components. |yes|

**Item Additional Properties:** not allowed **Example**

```yaml
- annotation: {}
  attributes: {}
  exposure: public
  protocol: http

```


##### components\[\]\.container\.endpoints\[\]\.annotation: object

Annotations to be added to Kubernetes Ingress or Openshift Route


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


##### components\[\]\.container\.endpoints\[\]\.attributes: object

Map of implementation-dependant string-based free-form attributes.

Examples of Che-specific attributes:
- cookiesAuthEnabled: "true" / "false",
- type: "terminal" / "ide" / "ide-dev",


**Additional Properties:** allowed 
#### components\[\]\.container\.env\[\]: array

Environment variables used in this container.

The following variables are reserved and cannot be overridden via env:

 - `$PROJECTS_ROOT`

 - `$PROJECT_SOURCE`


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**name**|`string`||yes|
|**value**|`string`||yes|

**Item Additional Properties:** not allowed **Example**

```yaml
- {}

```


#### components\[\]\.container\.volumeMounts\[\]: array

List of volumes mounts that should be mounted is this container.


**Items**


Volume that should be mounted to a component container

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**name**|`string`|The volume mount name is the name of an existing `Volume` component. If several containers mount the same volume name then they will reuse the same volume and will be able to access to the same files. Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|The path in the component container where the volume should be mounted. If not path is mentioned, default path is the is `/\<name\>`. |no|

**Item Additional Properties:** not allowed **Example**

```yaml
- {}

```


### components\[\]\.image: object

Allows specifying the definition of an image for outer loop builds


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**autoBuild**|`boolean`|Defines if the image should be built during startup.
Default value is `false` |no|
|[**dockerfile**](#componentsimagedockerfile)|`object`|Allows specifying dockerfile type build |no|
|**imageName**|`string`|Name of the image for the resulting outerloop build |yes|

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * dockerfile

**Example**

```yaml
dockerfile:
  devfileRegistry: {}
  git:
    checkoutFrom: {}
    remotes: {}

```


#### components\[\]\.image\.dockerfile: object

Allows specifying dockerfile type build


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**args**](#componentsimagedockerfileargs)|`string[]`|The arguments to supply to the dockerfile build. ||
|**buildContext**|`string`|Path of source directory to establish build context. Defaults to ${PROJECT_SOURCE} in the container ||
|[**devfileRegistry**](#componentsimagedockerfiledevfileregistry)|`object`|Dockerfile's Devfile Registry source |yes|
|[**git**](#componentsimagedockerfilegit)|`object`|Dockerfile's Git source |yes|
|**rootRequired**|`boolean`|Specify if a privileged builder pod is required.
Default value is `false` ||
|**uri**|`string`|URI Reference of a Dockerfile. It can be a full URL or a relative URI from the current devfile as the base URI. ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * uri

  **Option 2 (alternative):** 
**Required Properties:**

  * devfileRegistry

  **Option 3 (alternative):** 
**Required Properties:**

  * git

**Example**

```yaml
devfileRegistry: {}
git:
  checkoutFrom: {}
  remotes: {}

```


##### components\[\]\.image\.dockerfile\.args\[\]: array

The arguments to supply to the dockerfile build.


**Items**

**Item Type:** `string` 
##### components\[\]\.image\.dockerfile\.devfileRegistry: object

Dockerfile's Devfile Registry source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**id**|`string`|Id in a devfile registry that contains a Dockerfile. The src in the OCI registry required for the Dockerfile build will be downloaded for building the image. |yes|
|**registryUrl**|`string`|Devfile Registry URL to pull the Dockerfile from when using the Devfile Registry as Dockerfile src. To ensure the Dockerfile gets resolved consistently in different environments, it is recommended to always specify the `devfileRegistryUrl` when `Id` is used. |no|

**Additional Properties:** not allowed 
##### components\[\]\.image\.dockerfile\.git: object

Dockerfile's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#componentsimagedockerfilegitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured |no|
|**fileLocation**|`string`|Location of the Dockerfile in the Git repository when using git as Dockerfile src. Defaults to Dockerfile. |no|
|[**remotes**](#componentsimagedockerfilegitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. |yes|

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


###### components\[\]\.image\.dockerfile\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
###### components\[\]\.image\.dockerfile\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


### components\[\]\.kubernetes: object

Allows importing into the devworkspace the Kubernetes resources defined in a given manifest. For example this allows reusing the Kubernetes definitions used to deploy some runtime components in production.


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**deployByDefault**|`boolean`|Defines if the component should be deployed during startup.
Default value is `false` ||
|[**endpoints**](#componentskubernetesendpoints)|`object[]`|||
|**inlined**|`string`|Inlined manifest ||
|**uri**|`string`|Location in a file fetched from a uri. ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * uri

  **Option 2 (alternative):** 
**Required Properties:**

  * inlined

**Example**

```yaml
endpoints:
  - annotation: {}
    attributes: {}
    exposure: public
    protocol: http

```


#### components\[\]\.kubernetes\.endpoints\[\]: array

**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#componentskubernetesendpointsannotation)|`object`|Annotations to be added to Kubernetes Ingress or Openshift Route |no|
|[**attributes**](#componentskubernetesendpointsattributes)|`object`|Map of implementation-dependant string-based free-form attributes. |no|
|**exposure**|`string`|Describes how the endpoint should be exposed on the network. - `public` means that the endpoint will be exposed on the public network, typically through a K8S ingress or an OpenShift route. - `internal` means that the endpoint will be exposed internally outside of the main devworkspace POD, typically by K8S services, to be consumed by other elements running on the same cloud internal network. - `none` means that the endpoint will not be exposed and will only be accessible inside the main devworkspace POD, on a local address.
Default value is `public` Default: `"public"` Enum: `"public"`, `"internal"`, `"none"` |no|
|**name**|`string`|Maximal Length: `15` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|Path of the endpoint URL |no|
|**protocol**|`string`|Describes the application and transport protocols of the traffic that will go through this endpoint. - `http`: Endpoint will have `http` traffic, typically on a TCP connection. It will be automaticaly promoted to `https` when the `secure` field is set to `true`. - `https`: Endpoint will have `https` traffic, typically on a TCP connection. - `ws`: Endpoint will have `ws` traffic, typically on a TCP connection. It will be automaticaly promoted to `wss` when the `secure` field is set to `true`. - `wss`: Endpoint will have `wss` traffic, typically on a TCP connection. - `tcp`: Endpoint will have traffic on a TCP connection, without specifying an application protocol. - `udp`: Endpoint will have traffic on an UDP connection, without specifying an application protocol.
Default value is `http` Default: `"http"` Enum: `"http"`, `"https"`, `"ws"`, `"wss"`, `"tcp"`, `"udp"` |no|
|**secure**|`boolean`|Describes whether the endpoint should be secured and protected by some authentication process. This requires a protocol of `https` or `wss`. |no|
|**targetPort**|`integer`|Port number to be used within the container component. The same port cannot be used by two different container components. |yes|

**Item Additional Properties:** not allowed **Example**

```yaml
- annotation: {}
  attributes: {}
  exposure: public
  protocol: http

```


##### components\[\]\.kubernetes\.endpoints\[\]\.annotation: object

Annotations to be added to Kubernetes Ingress or Openshift Route


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


##### components\[\]\.kubernetes\.endpoints\[\]\.attributes: object

Map of implementation-dependant string-based free-form attributes.

Examples of Che-specific attributes:
- cookiesAuthEnabled: "true" / "false",
- type: "terminal" / "ide" / "ide-dev",


**Additional Properties:** allowed 
### components\[\]\.openshift: object

Allows importing into the devworkspace the OpenShift resources defined in a given manifest. For example this allows reusing the OpenShift definitions used to deploy some runtime components in production.


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**deployByDefault**|`boolean`|Defines if the component should be deployed during startup.
Default value is `false` ||
|[**endpoints**](#componentsopenshiftendpoints)|`object[]`|||
|**inlined**|`string`|Inlined manifest ||
|**uri**|`string`|Location in a file fetched from a uri. ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * uri

  **Option 2 (alternative):** 
**Required Properties:**

  * inlined

**Example**

```yaml
endpoints:
  - annotation: {}
    attributes: {}
    exposure: public
    protocol: http

```


#### components\[\]\.openshift\.endpoints\[\]: array

**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#componentsopenshiftendpointsannotation)|`object`|Annotations to be added to Kubernetes Ingress or Openshift Route |no|
|[**attributes**](#componentsopenshiftendpointsattributes)|`object`|Map of implementation-dependant string-based free-form attributes. |no|
|**exposure**|`string`|Describes how the endpoint should be exposed on the network. - `public` means that the endpoint will be exposed on the public network, typically through a K8S ingress or an OpenShift route. - `internal` means that the endpoint will be exposed internally outside of the main devworkspace POD, typically by K8S services, to be consumed by other elements running on the same cloud internal network. - `none` means that the endpoint will not be exposed and will only be accessible inside the main devworkspace POD, on a local address.
Default value is `public` Default: `"public"` Enum: `"public"`, `"internal"`, `"none"` |no|
|**name**|`string`|Maximal Length: `15` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|Path of the endpoint URL |no|
|**protocol**|`string`|Describes the application and transport protocols of the traffic that will go through this endpoint. - `http`: Endpoint will have `http` traffic, typically on a TCP connection. It will be automaticaly promoted to `https` when the `secure` field is set to `true`. - `https`: Endpoint will have `https` traffic, typically on a TCP connection. - `ws`: Endpoint will have `ws` traffic, typically on a TCP connection. It will be automaticaly promoted to `wss` when the `secure` field is set to `true`. - `wss`: Endpoint will have `wss` traffic, typically on a TCP connection. - `tcp`: Endpoint will have traffic on a TCP connection, without specifying an application protocol. - `udp`: Endpoint will have traffic on an UDP connection, without specifying an application protocol.
Default value is `http` Default: `"http"` Enum: `"http"`, `"https"`, `"ws"`, `"wss"`, `"tcp"`, `"udp"` |no|
|**secure**|`boolean`|Describes whether the endpoint should be secured and protected by some authentication process. This requires a protocol of `https` or `wss`. |no|
|**targetPort**|`integer`|Port number to be used within the container component. The same port cannot be used by two different container components. |yes|

**Item Additional Properties:** not allowed **Example**

```yaml
- annotation: {}
  attributes: {}
  exposure: public
  protocol: http

```


##### components\[\]\.openshift\.endpoints\[\]\.annotation: object

Annotations to be added to Kubernetes Ingress or Openshift Route


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


##### components\[\]\.openshift\.endpoints\[\]\.attributes: object

Map of implementation-dependant string-based free-form attributes.

Examples of Che-specific attributes:
- cookiesAuthEnabled: "true" / "false",
- type: "terminal" / "ide" / "ide-dev",


**Additional Properties:** allowed 
### components\[\]\.volume: object

Allows specifying the definition of a volume shared by several other components


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**ephemeral**|`boolean`|Ephemeral volumes are not stored persistently across restarts. Defaults to false ||
|**size**|`string`|Size of the volume ||

**Additional Properties:** not allowed 
## dependentProjects\[\]: array

Additional projects related to the main project in the devfile, contianing names and sources locations


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#dependentprojectsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|**clonePath**|`string`|Path relative to the root of the projects to which this project should be cloned into. This is a unix-style relative path (i.e. uses forward slashes). The path is invalid if it is absolute or tries to escape the project root through the usage of '..'. If not specified, defaults to the project name. |no|
|[**git**](#dependentprojectsgit)|`object`|Project's Git source |yes|
|**name**|`string`|Project name Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|[**zip**](#dependentprojectszip)|`object`|Project's Zip source |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * git

  **Option 2 (alternative):** 
**Required Properties:**

  * zip

**Example**

```yaml
- attributes: {}
  git:
    checkoutFrom: {}
    remotes: {}
  zip: {}

```


### dependentProjects\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
### dependentProjects\[\]\.git: object

Project's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#dependentprojectsgitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured |no|
|[**remotes**](#dependentprojectsgitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. |yes|

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


#### dependentProjects\[\]\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
#### dependentProjects\[\]\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


### dependentProjects\[\]\.zip: object

Project's Zip source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**location**|`string`|Zip project's source location address. Should be file path of the archive, e.g. file://$FILE_PATH ||

**Additional Properties:** not allowed 
## events: object

Bindings of commands to events. Each command is referred-to by its name.


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**postStart**](#eventspoststart)|`string[]`|IDs of commands that should be executed after the devworkspace is completely started. In the case of Che-Theia, these commands should be executed after all plugins and extensions have started, including project cloning. This means that those commands are not triggered until the user opens the IDE in his browser. ||
|[**postStop**](#eventspoststop)|`string[]`|IDs of commands that should be executed after stopping the devworkspace. ||
|[**preStart**](#eventsprestart)|`string[]`|IDs of commands that should be executed before the devworkspace start. Kubernetes-wise, these commands would typically be executed in init containers of the devworkspace POD. ||
|[**preStop**](#eventsprestop)|`string[]`|IDs of commands that should be executed before stopping the devworkspace. ||

**Additional Properties:** not allowed 
### events\.postStart\[\]: array

IDs of commands that should be executed after the devworkspace is completely started. In the case of Che-Theia, these commands should be executed after all plugins and extensions have started, including project cloning. This means that those commands are not triggered until the user opens the IDE in his browser.


**Items**

**Item Type:** `string` 
### events\.postStop\[\]: array

IDs of commands that should be executed after stopping the devworkspace.


**Items**

**Item Type:** `string` 
### events\.preStart\[\]: array

IDs of commands that should be executed before the devworkspace start. Kubernetes-wise, these commands would typically be executed in init containers of the devworkspace POD.


**Items**

**Item Type:** `string` 
### events\.preStop\[\]: array

IDs of commands that should be executed before stopping the devworkspace.


**Items**

**Item Type:** `string` 
## metadata: object

Optional metadata


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**architectures**](#metadataarchitectures)|`string[]`|Optional list of processor architectures that the devfile supports, empty list suggests that the devfile can be used on any architecture ||
|[**attributes**](#metadataattributes)|`object`|Map of implementation-dependant free-form YAML attributes. Deprecated, use the top-level attributes field instead. ||
|**description**|`string`|Optional devfile description ||
|**displayName**|`string`|Optional devfile display name ||
|**globalMemoryLimit**|`string`|Optional devfile global memory limit ||
|**icon**|`string`|Optional devfile icon, can be a URI or a relative path in the project ||
|**language**|`string`|Optional devfile language ||
|**name**|`string`|Optional devfile name ||
|**projectType**|`string`|Optional devfile project type ||
|**provider**|`string`|Optional devfile provider information ||
|**supportUrl**|`string`|Optional link to a page that provides support information ||
|[**tags**](#metadatatags)|`string[]`|Optional devfile tags ||
|**version**|`string`|Optional semver-compatible version Pattern: `^([0-9]+)\.([0-9]+)\.([0-9]+)(\-[0-9a-z-]+(\.[0-9a-z-]+)*)?(\+[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?$` ||
|**website**|`string`|Optional devfile website ||

**Additional Properties:** allowed **Example**

```yaml
attributes: {}

```


### metadata\.architectures\[\]: array

Optional list of processor architectures that the devfile supports, empty list suggests that the devfile can be used on any architecture


**Items**


Architecture describes the architecture type

**Item Type:** `string` **Item Enum:** `"amd64"`, `"arm64"`, `"ppc64le"`, `"s390x"` **Unique Items:** yes 
### metadata\.attributes: object

Map of implementation-dependant free-form YAML attributes. Deprecated, use the top-level attributes field instead.


**Additional Properties:** allowed 
### metadata\.tags\[\]: array

Optional devfile tags


**Items**

**Item Type:** `string` 
## parent: object

Parent devworkspace template


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#parentattributes)|`object`|Overrides of attributes encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules. ||
|[**commands**](#parentcommands)|`object[]`|Overrides of commands encapsulated in a parent devfile or a plugin. Overriding is done according to K8S strategic merge patch standard rules. ||
|[**components**](#parentcomponents)|`object[]`|Overrides of components encapsulated in a parent devfile or a plugin. Overriding is done according to K8S strategic merge patch standard rules. ||
|[**dependentProjects**](#parentdependentprojects)|`object[]`|Overrides of dependentProjects encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules. ||
|**id**|`string`|Id in a registry that contains a Devfile yaml file ||
|[**kubernetes**](#parentkubernetes)|`object`|Reference to a Kubernetes CRD of type DevWorkspaceTemplate |yes|
|[**projects**](#parentprojects)|`object[]`|Overrides of projects encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules. ||
|**registryUrl**|`string`|Registry URL to pull the parent devfile from when using id in the parent reference. To ensure the parent devfile gets resolved consistently in different environments, it is recommended to always specify the `registryUrl` when `id` is used. ||
|[**starterProjects**](#parentstarterprojects)|`object[]`|Overrides of starterProjects encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules. ||
|**uri**|`string`|URI Reference of a parent devfile YAML file. It can be a full URL or a relative URI with the current devfile as the base URI. ||
|[**variables**](#parentvariables)|`object`|Overrides of variables encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules. ||
|**version**|`string`|Specific stack/sample version to pull the parent devfile from, when using id in the parent reference. To specify `version`, `id` must be defined and used as the import reference source. `version` can be either a specific stack version, or `latest`. If no `version` specified, default version will be used. Pattern: `^(latest)\|(([1-9])\.([0-9]+)\.([0-9]+)(\-[0-9a-z-]+(\.[0-9a-z-]+)*)?(\+[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?)$` ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * uri

  **Option 2 (alternative):** 
**Required Properties:**

  * id

  **Option 3 (alternative):** 
**Required Properties:**

  * kubernetes

**Example**

```yaml
attributes: {}
commands:
  - apply:
      group: {}
    attributes: {}
    composite:
      group: {}
    exec:
      env:
        - {}
      group: {}
components:
  - attributes: {}
    container:
      annotation:
        deployment: {}
        service: {}
      endpoints:
        - annotation: {}
          attributes: {}
      env:
        - {}
      volumeMounts:
        - {}
    image:
      dockerfile:
        devfileRegistry: {}
        git:
          checkoutFrom: {}
          remotes: {}
    kubernetes:
      endpoints:
        - annotation: {}
          attributes: {}
    openshift:
      endpoints:
        - annotation: {}
          attributes: {}
    volume: {}
dependentProjects:
  - attributes: {}
    git:
      checkoutFrom: {}
      remotes: {}
    zip: {}
kubernetes: {}
projects:
  - attributes: {}
    git:
      checkoutFrom: {}
      remotes: {}
    zip: {}
starterProjects:
  - attributes: {}
    git:
      checkoutFrom: {}
      remotes: {}
    zip: {}
variables: {}

```


### parent\.attributes: object

Overrides of attributes encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules.


**Additional Properties:** allowed 
### parent\.commands\[\]: array

Overrides of commands encapsulated in a parent devfile or a plugin. Overriding is done according to K8S strategic merge patch standard rules.


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**apply**](#parentcommandsapply)|`object`|Command that consists in applying a given component definition, typically bound to a devworkspace event. |no|
|[**attributes**](#parentcommandsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|[**composite**](#parentcommandscomposite)|`object`|Composite command that allows executing several sub-commands either sequentially or concurrently |no|
|[**exec**](#parentcommandsexec)|`object`|CLI Command executed in an existing component container |no|
|**id**|`string`|Mandatory identifier that allows referencing this command in composite commands, from a parent, or in events. Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * exec

  **Option 2 (alternative):** 
**Required Properties:**

  * apply

  **Option 3 (alternative):** 
**Required Properties:**

  * composite

**Example**

```yaml
- apply:
    group: {}
  attributes: {}
  composite:
    group: {}
  exec:
    env:
      - {}
    group: {}

```


#### parent\.commands\[\]\.apply: object

Command that consists in applying a given component definition, typically bound to a devworkspace event.

For example, when an `apply` command is bound to a `preStart` event, and references a `container` component, it will start the container as a K8S initContainer in the devworkspace POD, unless the component has its `dedicatedPod` field set to `true`.

When no `apply` command exist for a given component, it is assumed the component will be applied at devworkspace start by default, unless `deployByDefault` for that component is set to false.


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**component**|`string`|Describes component that will be applied ||
|[**group**](#parentcommandsapplygroup)|`object`|Defines the group this command is part of ||
|**label**|`string`|Optional label that provides a label for this command to be used in Editor UI menus for example ||

**Additional Properties:** not allowed **Example**

```yaml
group: {}

```


##### parent\.commands\[\]\.apply\.group: object

Defines the group this command is part of


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**isDefault**|`boolean`|Identifies the default command for a given group kind ||
|**kind**|`string`|Kind of group the command is part of Enum: `"build"`, `"run"`, `"test"`, `"debug"`, `"deploy"` ||

**Additional Properties:** not allowed 
#### parent\.commands\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
#### parent\.commands\[\]\.composite: object

Composite command that allows executing several sub-commands either sequentially or concurrently


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**commands**](#parentcommandscompositecommands)|`string[]`|The commands that comprise this composite command ||
|[**group**](#parentcommandscompositegroup)|`object`|Defines the group this command is part of ||
|**label**|`string`|Optional label that provides a label for this command to be used in Editor UI menus for example ||
|**parallel**|`boolean`|Indicates if the sub-commands should be executed concurrently ||

**Additional Properties:** not allowed **Example**

```yaml
group: {}

```


##### parent\.commands\[\]\.composite\.commands\[\]: array

The commands that comprise this composite command


**Items**

**Item Type:** `string` 
##### parent\.commands\[\]\.composite\.group: object

Defines the group this command is part of


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**isDefault**|`boolean`|Identifies the default command for a given group kind ||
|**kind**|`string`|Kind of group the command is part of Enum: `"build"`, `"run"`, `"test"`, `"debug"`, `"deploy"` ||

**Additional Properties:** not allowed 
#### parent\.commands\[\]\.exec: object

CLI Command executed in an existing component container


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**commandLine**|`string`|The actual command-line string
Special variables that can be used:
- `$PROJECTS_ROOT`: A path where projects sources are mounted as defined by container component's sourceMapping.
- `$PROJECT_SOURCE`: A path to a project source ($PROJECTS_ROOT/\<project-name\>). If there are multiple projects, this will point to the directory of the first one. ||
|**component**|`string`|Describes component to which given action relates ||
|[**env**](#parentcommandsexecenv)|`object[]`|Optional list of environment variables that have to be set before running the command ||
|[**group**](#parentcommandsexecgroup)|`object`|Defines the group this command is part of ||
|**hotReloadCapable**|`boolean`|Specify whether the command is restarted or not when the source code changes. If set to `true` the command won't be restarted. A *hotReloadCapable* `run` or `debug` command is expected to handle file changes on its own and won't be restarted. A *hotReloadCapable* `build` command is expected to be executed only once and won't be executed again. This field is taken into account only for commands `build`, `run` and `debug` with `isDefault` set to `true`.
Default value is `false` ||
|**label**|`string`|Optional label that provides a label for this command to be used in Editor UI menus for example ||
|**workingDir**|`string`|Working directory where the command should be executed
Special variables that can be used:
- `$PROJECTS_ROOT`: A path where projects sources are mounted as defined by container component's sourceMapping.
- `$PROJECT_SOURCE`: A path to a project source ($PROJECTS_ROOT/\<project-name\>). If there are multiple projects, this will point to the directory of the first one. ||

**Additional Properties:** not allowed **Example**

```yaml
env:
  - {}
group: {}

```


##### parent\.commands\[\]\.exec\.env\[\]: array

Optional list of environment variables that have to be set before running the command


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**name**|`string`||yes|
|**value**|`string`||no|

**Item Additional Properties:** not allowed **Example**

```yaml
- {}

```


##### parent\.commands\[\]\.exec\.group: object

Defines the group this command is part of


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**isDefault**|`boolean`|Identifies the default command for a given group kind ||
|**kind**|`string`|Kind of group the command is part of Enum: `"build"`, `"run"`, `"test"`, `"debug"`, `"deploy"` ||

**Additional Properties:** not allowed 
### parent\.components\[\]: array

Overrides of components encapsulated in a parent devfile or a plugin. Overriding is done according to K8S strategic merge patch standard rules.


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#parentcomponentsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|[**container**](#parentcomponentscontainer)|`object`|Allows adding and configuring devworkspace-related containers |no|
|[**image**](#parentcomponentsimage)|`object`|Allows specifying the definition of an image for outer loop builds |no|
|[**kubernetes**](#parentcomponentskubernetes)|`object`|Allows importing into the devworkspace the Kubernetes resources defined in a given manifest. For example this allows reusing the Kubernetes definitions used to deploy some runtime components in production. |no|
|**name**|`string`|Mandatory name that allows referencing the component from other elements (such as commands) or from an external devfile that may reference this component through a parent or a plugin. Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|[**openshift**](#parentcomponentsopenshift)|`object`|Allows importing into the devworkspace the OpenShift resources defined in a given manifest. For example this allows reusing the OpenShift definitions used to deploy some runtime components in production. |no|
|[**volume**](#parentcomponentsvolume)|`object`|Allows specifying the definition of a volume shared by several other components |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * container

  **Option 2 (alternative):** 
**Required Properties:**

  * kubernetes

  **Option 3 (alternative):** 
**Required Properties:**

  * openshift

  **Option 4 (alternative):** 
**Required Properties:**

  * volume

  **Option 5 (alternative):** 
**Required Properties:**

  * image

**Example**

```yaml
- attributes: {}
  container:
    annotation:
      deployment: {}
      service: {}
    endpoints:
      - annotation: {}
        attributes: {}
    env:
      - {}
    volumeMounts:
      - {}
  image:
    dockerfile:
      devfileRegistry: {}
      git:
        checkoutFrom: {}
        remotes: {}
  kubernetes:
    endpoints:
      - annotation: {}
        attributes: {}
  openshift:
    endpoints:
      - annotation: {}
        attributes: {}
  volume: {}

```


#### parent\.components\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
#### parent\.components\[\]\.container: object

Allows adding and configuring devworkspace-related containers


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#parentcomponentscontainerannotation)|`object`|Annotations that should be added to specific resources for this container ||
|[**args**](#parentcomponentscontainerargs)|`string[]`|The arguments to supply to the command running the dockerimage component. The arguments are supplied either to the default command provided in the image or to the overridden command. ||
|[**command**](#parentcomponentscontainercommand)|`string[]`|The command to run in the dockerimage component instead of the default one provided in the image. ||
|**cpuLimit**|`string`|||
|**cpuRequest**|`string`|||
|**dedicatedPod**|`boolean`|Specify if a container should run in its own separated pod, instead of running as part of the main development environment pod.
Default value is `false` ||
|[**endpoints**](#parentcomponentscontainerendpoints)|`object[]`|||
|[**env**](#parentcomponentscontainerenv)|`object[]`|Environment variables used in this container. ||
|**image**|`string`|||
|**memoryLimit**|`string`|||
|**memoryRequest**|`string`|||
|**mountSources**|`boolean`|Toggles whether or not the project source code should be mounted in the component.
Defaults to true for all component types except plugins and components that set `dedicatedPod` to true. ||
|**sourceMapping**|`string`|Optional specification of the path in the container where project sources should be transferred/mounted when `mountSources` is `true`. When omitted, the default value of /projects is used. ||
|[**volumeMounts**](#parentcomponentscontainervolumemounts)|`object[]`|List of volumes mounts that should be mounted is this container. ||

**Additional Properties:** not allowed **Example**

```yaml
annotation:
  deployment: {}
  service: {}
endpoints:
  - annotation: {}
    attributes: {}
env:
  - {}
volumeMounts:
  - {}

```


##### parent\.components\[\]\.container\.annotation: object

Annotations that should be added to specific resources for this container


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**deployment**](#parentcomponentscontainerannotationdeployment)|`object`|Annotations to be added to deployment ||
|[**service**](#parentcomponentscontainerannotationservice)|`object`|Annotations to be added to service ||

**Additional Properties:** not allowed **Example**

```yaml
deployment: {}
service: {}

```


###### parent\.components\[\]\.container\.annotation\.deployment: object

Annotations to be added to deployment


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


###### parent\.components\[\]\.container\.annotation\.service: object

Annotations to be added to service


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


##### parent\.components\[\]\.container\.args\[\]: array

The arguments to supply to the command running the dockerimage component. The arguments are supplied either to the default command provided in the image or to the overridden command.

Defaults to an empty array, meaning use whatever is defined in the image.


**Items**

**Item Type:** `string` 
##### parent\.components\[\]\.container\.command\[\]: array

The command to run in the dockerimage component instead of the default one provided in the image.

Defaults to an empty array, meaning use whatever is defined in the image.


**Items**

**Item Type:** `string` 
##### parent\.components\[\]\.container\.endpoints\[\]: array

**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#parentcomponentscontainerendpointsannotation)|`object`|Annotations to be added to Kubernetes Ingress or Openshift Route |no|
|[**attributes**](#parentcomponentscontainerendpointsattributes)|`object`|Map of implementation-dependant string-based free-form attributes. |no|
|**exposure**|`string`|Describes how the endpoint should be exposed on the network. - `public` means that the endpoint will be exposed on the public network, typically through a K8S ingress or an OpenShift route. - `internal` means that the endpoint will be exposed internally outside of the main devworkspace POD, typically by K8S services, to be consumed by other elements running on the same cloud internal network. - `none` means that the endpoint will not be exposed and will only be accessible inside the main devworkspace POD, on a local address.
Default value is `public` Enum: `"public"`, `"internal"`, `"none"` |no|
|**name**|`string`|Maximal Length: `15` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|Path of the endpoint URL |no|
|**protocol**|`string`|Describes the application and transport protocols of the traffic that will go through this endpoint. - `http`: Endpoint will have `http` traffic, typically on a TCP connection. It will be automaticaly promoted to `https` when the `secure` field is set to `true`. - `https`: Endpoint will have `https` traffic, typically on a TCP connection. - `ws`: Endpoint will have `ws` traffic, typically on a TCP connection. It will be automaticaly promoted to `wss` when the `secure` field is set to `true`. - `wss`: Endpoint will have `wss` traffic, typically on a TCP connection. - `tcp`: Endpoint will have traffic on a TCP connection, without specifying an application protocol. - `udp`: Endpoint will have traffic on an UDP connection, without specifying an application protocol.
Default value is `http` Enum: `"http"`, `"https"`, `"ws"`, `"wss"`, `"tcp"`, `"udp"` |no|
|**secure**|`boolean`|Describes whether the endpoint should be secured and protected by some authentication process. This requires a protocol of `https` or `wss`. |no|
|**targetPort**|`integer`|Port number to be used within the container component. The same port cannot be used by two different container components. |no|

**Item Additional Properties:** not allowed **Example**

```yaml
- annotation: {}
  attributes: {}

```


###### parent\.components\[\]\.container\.endpoints\[\]\.annotation: object

Annotations to be added to Kubernetes Ingress or Openshift Route


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


###### parent\.components\[\]\.container\.endpoints\[\]\.attributes: object

Map of implementation-dependant string-based free-form attributes.

Examples of Che-specific attributes:
- cookiesAuthEnabled: "true" / "false",
- type: "terminal" / "ide" / "ide-dev",


**Additional Properties:** allowed 
##### parent\.components\[\]\.container\.env\[\]: array

Environment variables used in this container.

The following variables are reserved and cannot be overridden via env:

 - `$PROJECTS_ROOT`

 - `$PROJECT_SOURCE`


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**name**|`string`||yes|
|**value**|`string`||no|

**Item Additional Properties:** not allowed **Example**

```yaml
- {}

```


##### parent\.components\[\]\.container\.volumeMounts\[\]: array

List of volumes mounts that should be mounted is this container.


**Items**


Volume that should be mounted to a component container

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**name**|`string`|The volume mount name is the name of an existing `Volume` component. If several containers mount the same volume name then they will reuse the same volume and will be able to access to the same files. Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|The path in the component container where the volume should be mounted. If not path is mentioned, default path is the is `/\<name\>`. |no|

**Item Additional Properties:** not allowed **Example**

```yaml
- {}

```


#### parent\.components\[\]\.image: object

Allows specifying the definition of an image for outer loop builds


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**autoBuild**|`boolean`|Defines if the image should be built during startup.
Default value is `false` ||
|[**dockerfile**](#parentcomponentsimagedockerfile)|`object`|Allows specifying dockerfile type build ||
|**imageName**|`string`|Name of the image for the resulting outerloop build ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * dockerfile

  **Option 2 (alternative):** 
**Required Properties:**

  * autoBuild

**Example**

```yaml
dockerfile:
  devfileRegistry: {}
  git:
    checkoutFrom: {}
    remotes: {}

```


##### parent\.components\[\]\.image\.dockerfile: object

Allows specifying dockerfile type build


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**args**](#parentcomponentsimagedockerfileargs)|`string[]`|The arguments to supply to the dockerfile build. ||
|**buildContext**|`string`|Path of source directory to establish build context. Defaults to ${PROJECT_SOURCE} in the container ||
|[**devfileRegistry**](#parentcomponentsimagedockerfiledevfileregistry)|`object`|Dockerfile's Devfile Registry source ||
|[**git**](#parentcomponentsimagedockerfilegit)|`object`|Dockerfile's Git source ||
|**rootRequired**|`boolean`|Specify if a privileged builder pod is required.
Default value is `false` ||
|**uri**|`string`|URI Reference of a Dockerfile. It can be a full URL or a relative URI from the current devfile as the base URI. ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * uri

  **Option 2 (alternative):** 
**Required Properties:**

  * devfileRegistry

  **Option 3 (alternative):** 
**Required Properties:**

  * git

**Example**

```yaml
devfileRegistry: {}
git:
  checkoutFrom: {}
  remotes: {}

```


###### parent\.components\[\]\.image\.dockerfile\.args\[\]: array

The arguments to supply to the dockerfile build.


**Items**

**Item Type:** `string` 
###### parent\.components\[\]\.image\.dockerfile\.devfileRegistry: object

Dockerfile's Devfile Registry source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**id**|`string`|Id in a devfile registry that contains a Dockerfile. The src in the OCI registry required for the Dockerfile build will be downloaded for building the image. ||
|**registryUrl**|`string`|Devfile Registry URL to pull the Dockerfile from when using the Devfile Registry as Dockerfile src. To ensure the Dockerfile gets resolved consistently in different environments, it is recommended to always specify the `devfileRegistryUrl` when `Id` is used. ||

**Additional Properties:** not allowed 
###### parent\.components\[\]\.image\.dockerfile\.git: object

Dockerfile's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#parentcomponentsimagedockerfilegitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured ||
|**fileLocation**|`string`|Location of the Dockerfile in the Git repository when using git as Dockerfile src. Defaults to Dockerfile. ||
|[**remotes**](#parentcomponentsimagedockerfilegitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. ||

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


####### parent\.components\[\]\.image\.dockerfile\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
####### parent\.components\[\]\.image\.dockerfile\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


#### parent\.components\[\]\.kubernetes: object

Allows importing into the devworkspace the Kubernetes resources defined in a given manifest. For example this allows reusing the Kubernetes definitions used to deploy some runtime components in production.


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**deployByDefault**|`boolean`|Defines if the component should be deployed during startup.
Default value is `false` ||
|[**endpoints**](#parentcomponentskubernetesendpoints)|`object[]`|||
|**inlined**|`string`|Inlined manifest ||
|**uri**|`string`|Location in a file fetched from a uri. ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * uri

  **Option 2 (alternative):** 
**Required Properties:**

  * inlined

**Example**

```yaml
endpoints:
  - annotation: {}
    attributes: {}

```


##### parent\.components\[\]\.kubernetes\.endpoints\[\]: array

**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#parentcomponentskubernetesendpointsannotation)|`object`|Annotations to be added to Kubernetes Ingress or Openshift Route |no|
|[**attributes**](#parentcomponentskubernetesendpointsattributes)|`object`|Map of implementation-dependant string-based free-form attributes. |no|
|**exposure**|`string`|Describes how the endpoint should be exposed on the network. - `public` means that the endpoint will be exposed on the public network, typically through a K8S ingress or an OpenShift route. - `internal` means that the endpoint will be exposed internally outside of the main devworkspace POD, typically by K8S services, to be consumed by other elements running on the same cloud internal network. - `none` means that the endpoint will not be exposed and will only be accessible inside the main devworkspace POD, on a local address.
Default value is `public` Enum: `"public"`, `"internal"`, `"none"` |no|
|**name**|`string`|Maximal Length: `15` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|Path of the endpoint URL |no|
|**protocol**|`string`|Describes the application and transport protocols of the traffic that will go through this endpoint. - `http`: Endpoint will have `http` traffic, typically on a TCP connection. It will be automaticaly promoted to `https` when the `secure` field is set to `true`. - `https`: Endpoint will have `https` traffic, typically on a TCP connection. - `ws`: Endpoint will have `ws` traffic, typically on a TCP connection. It will be automaticaly promoted to `wss` when the `secure` field is set to `true`. - `wss`: Endpoint will have `wss` traffic, typically on a TCP connection. - `tcp`: Endpoint will have traffic on a TCP connection, without specifying an application protocol. - `udp`: Endpoint will have traffic on an UDP connection, without specifying an application protocol.
Default value is `http` Enum: `"http"`, `"https"`, `"ws"`, `"wss"`, `"tcp"`, `"udp"` |no|
|**secure**|`boolean`|Describes whether the endpoint should be secured and protected by some authentication process. This requires a protocol of `https` or `wss`. |no|
|**targetPort**|`integer`|Port number to be used within the container component. The same port cannot be used by two different container components. |no|

**Item Additional Properties:** not allowed **Example**

```yaml
- annotation: {}
  attributes: {}

```


###### parent\.components\[\]\.kubernetes\.endpoints\[\]\.annotation: object

Annotations to be added to Kubernetes Ingress or Openshift Route


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


###### parent\.components\[\]\.kubernetes\.endpoints\[\]\.attributes: object

Map of implementation-dependant string-based free-form attributes.

Examples of Che-specific attributes:
- cookiesAuthEnabled: "true" / "false",
- type: "terminal" / "ide" / "ide-dev",


**Additional Properties:** allowed 
#### parent\.components\[\]\.openshift: object

Allows importing into the devworkspace the OpenShift resources defined in a given manifest. For example this allows reusing the OpenShift definitions used to deploy some runtime components in production.


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**deployByDefault**|`boolean`|Defines if the component should be deployed during startup.
Default value is `false` ||
|[**endpoints**](#parentcomponentsopenshiftendpoints)|`object[]`|||
|**inlined**|`string`|Inlined manifest ||
|**uri**|`string`|Location in a file fetched from a uri. ||

**Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * uri

  **Option 2 (alternative):** 
**Required Properties:**

  * inlined

**Example**

```yaml
endpoints:
  - annotation: {}
    attributes: {}

```


##### parent\.components\[\]\.openshift\.endpoints\[\]: array

**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**annotation**](#parentcomponentsopenshiftendpointsannotation)|`object`|Annotations to be added to Kubernetes Ingress or Openshift Route |no|
|[**attributes**](#parentcomponentsopenshiftendpointsattributes)|`object`|Map of implementation-dependant string-based free-form attributes. |no|
|**exposure**|`string`|Describes how the endpoint should be exposed on the network. - `public` means that the endpoint will be exposed on the public network, typically through a K8S ingress or an OpenShift route. - `internal` means that the endpoint will be exposed internally outside of the main devworkspace POD, typically by K8S services, to be consumed by other elements running on the same cloud internal network. - `none` means that the endpoint will not be exposed and will only be accessible inside the main devworkspace POD, on a local address.
Default value is `public` Enum: `"public"`, `"internal"`, `"none"` |no|
|**name**|`string`|Maximal Length: `15` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**path**|`string`|Path of the endpoint URL |no|
|**protocol**|`string`|Describes the application and transport protocols of the traffic that will go through this endpoint. - `http`: Endpoint will have `http` traffic, typically on a TCP connection. It will be automaticaly promoted to `https` when the `secure` field is set to `true`. - `https`: Endpoint will have `https` traffic, typically on a TCP connection. - `ws`: Endpoint will have `ws` traffic, typically on a TCP connection. It will be automaticaly promoted to `wss` when the `secure` field is set to `true`. - `wss`: Endpoint will have `wss` traffic, typically on a TCP connection. - `tcp`: Endpoint will have traffic on a TCP connection, without specifying an application protocol. - `udp`: Endpoint will have traffic on an UDP connection, without specifying an application protocol.
Default value is `http` Enum: `"http"`, `"https"`, `"ws"`, `"wss"`, `"tcp"`, `"udp"` |no|
|**secure**|`boolean`|Describes whether the endpoint should be secured and protected by some authentication process. This requires a protocol of `https` or `wss`. |no|
|**targetPort**|`integer`|Port number to be used within the container component. The same port cannot be used by two different container components. |no|

**Item Additional Properties:** not allowed **Example**

```yaml
- annotation: {}
  attributes: {}

```


###### parent\.components\[\]\.openshift\.endpoints\[\]\.annotation: object

Annotations to be added to Kubernetes Ingress or Openshift Route


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


###### parent\.components\[\]\.openshift\.endpoints\[\]\.attributes: object

Map of implementation-dependant string-based free-form attributes.

Examples of Che-specific attributes:
- cookiesAuthEnabled: "true" / "false",
- type: "terminal" / "ide" / "ide-dev",


**Additional Properties:** allowed 
#### parent\.components\[\]\.volume: object

Allows specifying the definition of a volume shared by several other components


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**ephemeral**|`boolean`|Ephemeral volumes are not stored persistently across restarts. Defaults to false ||
|**size**|`string`|Size of the volume ||

**Additional Properties:** not allowed 
### parent\.dependentProjects\[\]: array

Overrides of dependentProjects encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules.


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#parentdependentprojectsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|**clonePath**|`string`|Path relative to the root of the projects to which this project should be cloned into. This is a unix-style relative path (i.e. uses forward slashes). The path is invalid if it is absolute or tries to escape the project root through the usage of '..'. If not specified, defaults to the project name. |no|
|[**git**](#parentdependentprojectsgit)|`object`|Project's Git source |no|
|**name**|`string`|Project name Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|[**zip**](#parentdependentprojectszip)|`object`|Project's Zip source |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * git

  **Option 2 (alternative):** 
**Required Properties:**

  * zip

**Example**

```yaml
- attributes: {}
  git:
    checkoutFrom: {}
    remotes: {}
  zip: {}

```


#### parent\.dependentProjects\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
#### parent\.dependentProjects\[\]\.git: object

Project's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#parentdependentprojectsgitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured ||
|[**remotes**](#parentdependentprojectsgitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. ||

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


##### parent\.dependentProjects\[\]\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
##### parent\.dependentProjects\[\]\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


#### parent\.dependentProjects\[\]\.zip: object

Project's Zip source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**location**|`string`|Zip project's source location address. Should be file path of the archive, e.g. file://$FILE_PATH ||

**Additional Properties:** not allowed 
### parent\.kubernetes: object

Reference to a Kubernetes CRD of type DevWorkspaceTemplate


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**name**|`string`||yes|
|**namespace**|`string`||no|

**Additional Properties:** not allowed 
### parent\.projects\[\]: array

Overrides of projects encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules.


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#parentprojectsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|**clonePath**|`string`|Path relative to the root of the projects to which this project should be cloned into. This is a unix-style relative path (i.e. uses forward slashes). The path is invalid if it is absolute or tries to escape the project root through the usage of '..'. If not specified, defaults to the project name. |no|
|[**git**](#parentprojectsgit)|`object`|Project's Git source |no|
|**name**|`string`|Project name Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|[**zip**](#parentprojectszip)|`object`|Project's Zip source |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * git

  **Option 2 (alternative):** 
**Required Properties:**

  * zip

**Example**

```yaml
- attributes: {}
  git:
    checkoutFrom: {}
    remotes: {}
  zip: {}

```


#### parent\.projects\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
#### parent\.projects\[\]\.git: object

Project's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#parentprojectsgitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured ||
|[**remotes**](#parentprojectsgitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. ||

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


##### parent\.projects\[\]\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
##### parent\.projects\[\]\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


#### parent\.projects\[\]\.zip: object

Project's Zip source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**location**|`string`|Zip project's source location address. Should be file path of the archive, e.g. file://$FILE_PATH ||

**Additional Properties:** not allowed 
### parent\.starterProjects\[\]: array

Overrides of starterProjects encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules.


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#parentstarterprojectsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|**description**|`string`|Description of a starter project |no|
|[**git**](#parentstarterprojectsgit)|`object`|Project's Git source |no|
|**name**|`string`|Project name Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**subDir**|`string`|Sub-directory from a starter project to be used as root for starter project. |no|
|[**zip**](#parentstarterprojectszip)|`object`|Project's Zip source |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * git

  **Option 2 (alternative):** 
**Required Properties:**

  * zip

**Example**

```yaml
- attributes: {}
  git:
    checkoutFrom: {}
    remotes: {}
  zip: {}

```


#### parent\.starterProjects\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
#### parent\.starterProjects\[\]\.git: object

Project's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#parentstarterprojectsgitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured ||
|[**remotes**](#parentstarterprojectsgitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. ||

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


##### parent\.starterProjects\[\]\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
##### parent\.starterProjects\[\]\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


#### parent\.starterProjects\[\]\.zip: object

Project's Zip source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**location**|`string`|Zip project's source location address. Should be file path of the archive, e.g. file://$FILE_PATH ||

**Additional Properties:** not allowed 
### parent\.variables: object

Overrides of variables encapsulated in a parent devfile. Overriding is done according to K8S strategic merge patch standard rules.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


## projects\[\]: array

Projects worked on in the devworkspace, containing names and sources locations


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#projectsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|**clonePath**|`string`|Path relative to the root of the projects to which this project should be cloned into. This is a unix-style relative path (i.e. uses forward slashes). The path is invalid if it is absolute or tries to escape the project root through the usage of '..'. If not specified, defaults to the project name. |no|
|[**git**](#projectsgit)|`object`|Project's Git source |yes|
|**name**|`string`|Project name Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|[**zip**](#projectszip)|`object`|Project's Zip source |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * git

  **Option 2 (alternative):** 
**Required Properties:**

  * zip

**Example**

```yaml
- attributes: {}
  git:
    checkoutFrom: {}
    remotes: {}
  zip: {}

```


### projects\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
### projects\[\]\.git: object

Project's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#projectsgitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured |no|
|[**remotes**](#projectsgitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. |yes|

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


#### projects\[\]\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
#### projects\[\]\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


### projects\[\]\.zip: object

Project's Zip source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**location**|`string`|Zip project's source location address. Should be file path of the archive, e.g. file://$FILE_PATH ||

**Additional Properties:** not allowed 
## starterProjects\[\]: array

StarterProjects is a project that can be used as a starting point when bootstrapping new projects


**Items**

**Item Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**attributes**](#starterprojectsattributes)|`object`|Map of implementation-dependant free-form YAML attributes. |no|
|**description**|`string`|Description of a starter project |no|
|[**git**](#starterprojectsgit)|`object`|Project's Git source |yes|
|**name**|`string`|Project name Maximal Length: `63` Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$` |yes|
|**subDir**|`string`|Sub-directory from a starter project to be used as root for starter project. |no|
|[**zip**](#starterprojectszip)|`object`|Project's Zip source |no|

**Item Additional Properties:** not allowed   **Option 1 (alternative):** 
**Required Properties:**

  * git

  **Option 2 (alternative):** 
**Required Properties:**

  * zip

**Example**

```yaml
- attributes: {}
  git:
    checkoutFrom: {}
    remotes: {}
  zip: {}

```


### starterProjects\[\]\.attributes: object

Map of implementation-dependant free-form YAML attributes.


**Additional Properties:** allowed 
### starterProjects\[\]\.git: object

Project's Git source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|[**checkoutFrom**](#starterprojectsgitcheckoutfrom)|`object`|Defines from what the project should be checked out. Required if there are more than one remote configured |no|
|[**remotes**](#starterprojectsgitremotes)|`object`|The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured. |yes|

**Additional Properties:** not allowed **Example**

```yaml
checkoutFrom: {}
remotes: {}

```


#### starterProjects\[\]\.git\.checkoutFrom: object

Defines from what the project should be checked out. Required if there are more than one remote configured


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**remote**|`string`|The remote name should be used as init. Required if there are more than one remote configured ||
|**revision**|`string`|The revision to checkout from. Should be branch name, tag or commit id. Default branch is used if missing or specified revision is not found. ||

**Additional Properties:** not allowed 
#### starterProjects\[\]\.git\.remotes: object

The remotes map which should be initialized in the git project. Projects must have at least one remote configured while StarterProjects & Image Component's Git source can only have at most one remote configured.


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


### starterProjects\[\]\.zip: object

Project's Zip source


**Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**location**|`string`|Zip project's source location address. Should be file path of the archive, e.g. file://$FILE_PATH ||

**Additional Properties:** not allowed 
## variables: object

Map of key-value variables used for string replacement in the devfile. Values can be referenced via {{variable-key}} to replace the corresponding value in string fields in the devfile. Replacement cannot be used for

 - schemaVersion, metadata, parent source

 - element identifiers, e.g. command id, component name, endpoint name, project name

 - references to identifiers, e.g. in events, a command's component, container's volume mount name

 - string enums, e.g. command group kind, endpoint exposure


**Additional Properties**

|Name|Type|Description|Required|
|----|----|-----------|--------|
|**Additional Properties**|`string`|||


