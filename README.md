# Quarkus Helm Chart
A Helm chart for building and deploying a [Quarkus](https://quarkus.io/) application on OpenShift.

## Prerequisites
Below are prerequisites that may apply to your use case.

### Pull Secret
You will need to create a pull secret if you pull an S2I builder or Docker base image from an external registry. Use the following command as a reference to create your pull secret:
```bash
oc create secret docker-registry my-pull-secret --docker-server=$SERVER_URL --docker-username=$USERNAME --docker-password=$PASSWORD --docker-email=$EMAIL
```

You can use this secret by passing `--set build.pullSecret=my-pull-secret` to `helm install`, or you can configure this in a values file:
```yaml
build:
  pullSecret: my-pull-secret
```
and apply by passing `--values $VALUES_FILE`.

### Push Secret
You will need to create a push secret if you want to push your image to an external registry. Use the following command as a reference to create your push secret:
```bash
oc create secret docker-registry my-push-secret --docker-server=$SERVER_URL --docker-username=$USERNAME --docker-password=$PASSWORD --docker-email=$EMAIL
```

You can use this secret by passing `--set build.output.pushSecret=my-push-secret` and `--set build.output.kind=DockerImage` to `helm install`, or you can configure these in a values file:
```yaml
build:
  output:
    kind: DockerImage
    pushSecret: my-push-secret
```

## Values
Below is a table of each value used to configure this chart.

| Value | Description | Default | Additional Information |
| ----- | ----------- | ------- | ---------------------- |
| `image.name` | Name of the image you want to build/deploy | Defaults to the Helm release name. | The chart will create/reference an [ImageStream](https://docs.openshift.com/container-platform/4.6/openshift_images/image-streams-manage.html) based on this value. |
| `image.tag` | Tag that you want to build/deploy | `latest` | The chart will create/reference an [ImageStreamTag](https://docs.openshift.com/container-platform/4.6/openshift_images/image-streams-manage.html#images-using-imagestream-tags_image-streams-managing) based on the name provided |
| `build.enabled` | Determines if build-related resources should be created. | `true` | Set this to `false` if you want to deploy a previously built image. Leave this set to `true` if you want to build and deploy a new image. |
| `build.uri` | Git URI that references your git repo | https://github.com/redhat-developer-helm-quickstarts/quarkus-getting-started | This value defaults to a sample application. Be sure to override this if you want to build and deploy your own application. |
| `build.ref` | Git ref containing the application you want to build | main | - |
| `build.contextDir` | The sub-directory where the application source code exists | - | - |
| `build.mode` | Determines whether the Quarkus mode should be set to `jvm` or `native` | `jvm` | Options: `jvm` or `native` |
| `build.jvm.imageStreamTag.name` | The ImageStreamTag name of the desired builder image | `java:11` | Only has an effect if `build.mode` is set to `jvm` |
| `build.jvm.imageStreamTag.useReleaseNamespace` | Determines if the builder ImageStreamTag referenced by `build.jvm.imageStreamTag.name` is in the same namespace you are installing this Helm chart to | `false` | Only has an effect if `build.mode` is set to `jvm` |
| `build.jvm.imageStreamTag.namespace` | The namespace containing the builder ImageStreamTag | `openshift` | Only has an effect if `build.jvm.imageStreamTag.useReleaseNamespace` is `false` and `build.mode` is `jvm` |
| `build.native.useDefaultDockerfile` | Determines if Helm should generate a default Dockerfile to build a native binary | `true` | Only has an effect if `build.mode` is `native`. Set this to `false` if you would like to provide your own Dockerfile in git. |
| `build.native.dockerfilePath` | Path to the Dockerfile | Dockerfile | Path is relative to the context dir |
| `build.output.kind` | Determines if the image will be pushed to an ImageStreamTag or a DockerImage (external registry) | ImageStreamTag | More information: More information: https://docs.openshift.com/container-platform/4.6/builds/managing-build-output.html |
| `build.output.pushSecret` | Push secret name | - | Used only if build.output.kind == 'DockerImage' |
| `build.pullSecret` | Image pull secret | - | More information: https://docs.openshift.com/container-platform/4.6/openshift_images/managing_images/using-image-pull-secrets.html |
| `build.env` | Freeform `env` stanza | - | More information: https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/ |
| `build.resources` | Freeform `resources` stanza | - | More information: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/ |
| `deploy.replicas` | Number of pod replicas to deploy | `1` | - |
| `deploy.resources` | Freeform `resources` stanza | - | More information: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/ |
| `deploy.serviceType` | Type of service to create | `ClusterIP` | More information: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types |
| `deploy.ports` | Freeform service `ports` stanza. | See [values.yaml](./values.yaml) | More information: https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service |
| `deploy.route.enabled` | Determines if a Route should be created | `true` | Allows clients outside of OpenShift to access your application |
| `deploy.route.targetPort` | The port that the Route should target traffic to | `http` | - |
| `deploy.route.tls.enabled` | Determines if the Route should be TLS-encrypted | `true` | More information: https://docs.openshift.com/container-platform/4.6/networking/routes/secured-routes.html |
| `deploy.route.tls.termination` | Determines the type of TLS termination to use | `edge` | Options: `edge`, `reencrypt`, `passthrough` |
| `deploy.route.tls.insecureEdgeTerminationPolicy` | Determines if insecure traffic should be redirected | `Redirect` | Options: "Allow", "Disable", "Redirect" |
| `deploy.route.tls.key` | Provides key file contents | - | This is a secret. Do not check this value into git. |
| `deploy.route.tls.caCertificate` | Provides the cert authority certificate contents | - | - |
| `deploy.route.tls.certificate` | Provides certificate contents | - | - |
| `deploy.route.tls.destinationCACertificate` | Provides the destination CA Certificate for reencrypt routes | - | - |
| `deploy.livenessProbe` | Freeform `livenessProbe` stanza. | See [values.yaml](./values.yaml) | More information: https://docs.openshift.com/container-platform/4.6/applications/application-health.html#application-health-about_application-health |
| `deploy.readinessProbe` | Freeform `readinessProbe` stanza. | See [values.yaml](./values.yaml) | More information: https://docs.openshift.com/container-platform/4.6/applications/application-health.html#application-health-about_application-health |
| `deploy.env` | Freeform `env` stanza | - | More information: https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/ |
| `deploy.envFrom` | Freeform `envFrom` stanza | - | More information: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables |
| `deploy.applicationProperties.enabled` | Determines if application properties should be externalized in a ConfigMap | `false` | - |
| `deploy.applicationProperties.mountPath` | Location to mount the application.properties file | `/deployments/config/` | - |
| `deploy.applicationProperties.properties` | The application.properties file contents | - | - |
| `deploy.volumeMounts` | Freeform volume mounts | - | More information: https://kubernetes.io/docs/concepts/storage/volumes/ |
| `deploy.volumes` | Freeform volumes | - | More information: https://kubernetes.io/docs/concepts/storage/volumes/ |
| `deploy.initContainers` | Freeform init containers | - | More information: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/ |
| `deploy.extraContainers` | Freeform containers | - | More information: https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates |
| `global.nameOverride` | Overrides the release name | - | Resources are named after the release name. Set this value if you want to override the release name. |

### JVM vs Native Modes
This chart supports building Quarkus in both `jvm` and `native` modes.
* If `build.mode` is `jvm`, Helm will create a BuildConfig with `git` source and the [Source](https://docs.openshift.com/container-platform/4.6/builds/understanding-image-builds.html#build-strategy-s2i_understanding-image-builds) strategy.
* If `build.mode` is `native`, Helm will create a BuildConfig with `git` source and the [Docker](https://docs.openshift.com/container-platform/4.6/builds/understanding-image-builds.html#builds-strategy-docker-build_understanding-image-builds) strategy. Additionally, if `build.native.useDefaultDockerfile` is `true`, Helm will provide a default Dockerfile as a second input for building a native Quarkus binary. Below is the default Dockerfile that will be used:
  ```Dockerfile
  FROM registry.redhat.io/quarkus/mandrel-20-rhel8 AS builder
  WORKDIR /build/
  COPY . /build/
  RUN ./mvnw clean package -Pnative

  FROM registry.redhat.io/ubi8/ubi-minimal
  WORKDIR /deployments/
  COPY --from=builder /build/target/*-runner /deployments/application
  RUN chmod 110 /deployments/application
  CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
  ```
  If you wish to use the default Dockerfile, you must have the `mvnw` executable located in your project root. 

  If you wish to override the default Dockerfile, you can set `build.native.useDefaultDockerfile` to `false` and provide your own Dockerfile in your project root.

### Install OpenShift

```bash
oc login --token=$VALUE --server=$SERVER
helm delete quarkus-app
helm install quarkus-app quarkus-helm-chart/
```