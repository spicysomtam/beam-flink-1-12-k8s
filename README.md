# Introduction

Combine Apache Beam with an Apache Flink runner and cluster (kubernetes) using examples from both websites. 

Kubernetes cluster setup further down this document.

All of this runs on Linux (Ubuntu, etc).

Apache Beam was 2.28.0 (see [Apache Beam downloads](https://beam.apache.org/get-started/downloads/)).

# Walking through the Beam word-count example

This is all java 8 (openjdk worked for me; probably Oracle jdk will work fine).

## Java and maven setup

Setup described [in Beam docs](https://beam.apache.org/get-started/quickstart-java/); we use the `word-count` example to get things going.

Then we follow word-count [Getting Started](https://beam.apache.org/get-started/wordcount-example/).

## Using the default Beam runner to confirm all is good

Lets build and run using the default beam runner:
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts" -Pdirect-runner
```

## Using a downloaded Flink Mini Cluster runner

Then we can move onto to using a downloaded Flink mini cluster runner to confirm using the Flink runner is good. You don't need to do anything other than run this command:

```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--runner=FlinkRunner --inputFile=pom.xml --output=counts" -Pflink-runner
```
Logging:
```
Mar 26, 2021 5:30:31 PM org.apache.flink.runtime.minicluster.MiniCluster start
INFO: Starting Flink Mini Cluster
Mar 26, 2021 5:30:31 PM org.apache.flink.runtime.minicluster.MiniCluster start
INFO: Starting Metrics Registry
Mar 26, 2021 5:30:31 PM org.apache.flink.runtime.metrics.MetricRegistryImpl <init>
INFO: No metrics reporter configured, no metrics will be exposed/reported.
Mar 26, 2021 5:30:31 PM org.apache.flink.runtime.minicluster.MiniCluster start
INFO: Starting RPC Service(s)
```

## Use a Flink runner in a Flink cluster

Spin up a Flink cluster in kubernetes (see next section for instructions).

Setup a port forward to the cluster (might need to port forward to the job manager pod if you are using an older kubectl):

```
$ kubectl port-forward service/flink-jobmanager 8081:8081 &
```

Try and run against the flink cluster. We use `/etc/passwd` as its readable by everyone on the Job Manager container: 

```
$ mvn package exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
-Dexec.args="--runner=FlinkRunner --flinkMaster=localhost:8081 --filesToStage=target/word-count-beam-bundled-0.1.jar --inputFile=/etc/passwd --output=/tmp/counts" \
 -Pflink-runner -DskipTests
```

You can check the run in the gui in a web browser via the kube port forward using url http://localhost:8081. You will see 12 tasks finished (no failures).

A java invocation of the last maven command:

```
$ java -cp target/word-count-beam-bundled-0.1.jar \
  org.apache.beam.examples.WordCount \
  --runner=FlinkRunner \
  --flinkMaster=localhost:8081 \
  --filesToStage=target/word-count-beam-bundled-0.1.jar \
  --inputFile=/etc/passwd \
  --output=/tmp/count
```

## Fink cluster major versions that work with this release of Beam

If you check the kubernetes deployments in the `k8s` folder, you will see older versions of Flink commented out; this was an easy way to test compatibility with the `word-count` example. Note that the Job manager and Task manager versions should be the same. I have used the latest minor versions where there is an official docker image (official docker image means a good release; bad release docker images were probably pulled to discourage use of them).

| Flink version | Compatible? |
| ------------- | ----------- |
| 1.12.2        | yes         |
| 1.11.3        | yes         |
| 1.10.3        | no          |
| 1.9.3         | no          |
| 1.8.1         | no          |


# Flink kubernetes standalone session cluster setup

This is cut and paste from official Flink docs for a Standalone Session cluster that you can spin up in Kind/Minikube/microk8s. The example above worked perfectly for me using a [Kind kubernetes cluster](https://kind.sigs.k8s.io/docs/user/quick-start/) (Kubernetes in Docker). Kind is really easy to spin up if you already have docker running.

Office docs [here](https://ci.apache.org/projects/flink/flink-docs-release-1.12/deployment/resource-providers/standalone/kubernetes.html#starting-a-kubernetes-cluster-session-mode).

## Deploy 

Set your kube context, then:

```
kubectl apply -f k8s/
```

## Destroy

Set your kube context, then:

```
kubectl delete -f k8s/
```

