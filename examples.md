# Examples

The `examples` directory contains various examples of input, config and queries for the built-in Engines. They are often in the form of an integration test but should not be relied upon as such unless you know what you are doing.

For instance the UR integration tests will fail if you are not using a specific version of Elasticsearch. This is purely an artifact of the way the test works and does not indicate a problem. 

So treat "tests" as examples, not as something that must pass before use of the Engine.

# Universal Recommender (UR)

The UR tests are run from the harness-cli container or wherever it is installed. Each test targets a certain style of deployment. People usually start with the "localhost" version which runs on a dev machine or wherever the CLI and Harness Server are installed. It assumes:

 - `localhost:9090` is the address of the Harness Server
 - the Spark Master is localhost. The Spark settings are in the UR Engines JSON config which has a `sparkConf` section that sets `master="local"`

The flow of the test does the following:

 1. Delete any `test_ur` Engine Instance using `harness-cli delete test_ur`. If there is no Engine Instance by that name there will be a warning, ignore it if this is true.
 2. Add the `test_ur` with `harness-cli add ...`
 3. input an extremely small sample dataset using the Harness Python SDK and the Harness REST API. 
 4. trigger training of the `test_ur`
 5. sleep for some time to wait for the training, which executes a Spark Job. If the delay is too short, it should be lengthened. The Spark Job does the following:
    - read all data by connecting to Mongo
    - calculate the UR model from the sample data in Mongo
    - write the model to Elasticsearch, which the Engine's JSON points to. See the `spark.es.nodes` value for the Elasticsearch node address.
 6. once the delay is over, run queries by using `curl` in a script file. A potential problem is the delay may be too small for the Job to complete.
 7. collects results into an `actuall...` file
 8. compares `actuall...` to `examples/ur/expected...` file and fails if there are differences, which will be printed to the console.

This will exercise all parts of a workflow for the UR. If there are any problems we examine the contents of the DB, ES, and logs to discover the phase of workflow where the problem occurred. 

## Simple Integration Test

The simples example test is in `harness-cli/examples/ur/simple-integration-test.sh`

### All Localhost Services

The script defaults to assuming all service are running on the localhost except Spark, which will run inside the Harness process and so does not need to be installed.

This works well when using a debugger to change code in Engines or Harness. 

Run the script in the all-in-one OS level installation with:

```
./examples/ur/simple-integration-test.sh
```

When using a different deployment style&mdash;like Kubernetes, docker-compose containers, etc&mdash;use the following general rules

### Kubernetes

By nature you will run `harness-cli` remotely connected to the Harness server. You will have Harness mapped or "port forwarded" to `localhost:9090`. This will fit the assumptions in `simple-integration-test.sh`. Run the test: 

```
./examples/ur/simple-integration-test.sh k8s
```

From the Harness server's point of view the Kubernetes addresses for services will have to be used in the UR Engines config, particularly the `sparkConf` section. With the `k8s` flag the script will use the following engine config for Spark:

```
  "sparkConf": {
    "es.index.auto.create": "true",
    "es.nodes": "elasticsearch-client",
    "es.nodes.wan.only": "true",
    "master": "spark://spark-api:7077",
    "spark.driver.memory": "3g",
    "spark.es.index.auto.create": "true",
    "spark.es.nodes": "elasticsearch-client",
    "spark.es.nodes.wan.only": "true",
    "spark.executor.memory": "3g",
    "spark.kryo.referenceTracking": "false",
    "spark.kryo.registrator": "org.apache.mahout.sparkbindings.io.MahoutKryoRegistrator",
    "spark.kryoserializer.buffer": "300m",
    "spark.serializer": "org.apache.spark.serializer.KryoSerializer"
  },
```

This should work for an ActionML k8s installation. Change Elasticsearch and Spark `master` addresses to use the k8s service addresses if they are different.

### Docker-compose

This deployment assumes you use the ActionML docker-compose.yml. Start with instructions [here](README.md) Login to the `harness-cli` container to run the test. Using the ActionML docker-compose.yml and all the containers it specifies, use the following `sparkConf`.

```
  "sparkConf": {
    "es.index.auto.create": "true",
    "es.nodes": "elasticsearch",
    "es.nodes.wan.only": "true",
    "master": "local",
    "spark.driver.memory": "3g",
    "spark.es.index.auto.create": "true",
    "spark.es.nodes": "elasticsearch",
    "spark.es.nodes.wan.only": "true",
    "spark.executor.memory": "3g",
    "spark.kryo.referenceTracking": "false",
    "spark.kryo.registrator": "org.apache.mahout.sparkbindings.io.MahoutKryoRegistrator",
    "spark.kryoserializer.buffer": "300m",
    "spark.serializer": "org.apache.spark.serializer.KryoSerializer"
  },
```

The docker-compose.yml configures the CLI and Harness properly so nothing changes in the test script except the above `sparkConf`

## Troubleshooting

 - **Harness Log ERROR**: most ERRORs are reported in Harness logs. Check here for clues first. Other clues depend on the phase of the workflow where they occurred.
 - **Phase 1-2**: errors here may be connection related. Check that the CLI finds Harness by running `harness-cli status`
 - **Phase 3**: if there is trouble sending data, since this uses the Python SDK as does the CLI, run `harness-cli status`. To verify data has been input look in the DB in the db `test_ur` collection `events` using the Mongo shell.
 - **Phase 4-5**: Spark Job problems will usually show up in the Harness logs. To look deeper examine the contents of Elasticsearch by using curl to see indexes. The most recent `test_ur_...` index should have documents for each item in the sample data. These will be named `iPad Pro`, `Galaxy` etc. If no index has data, the Spark job was unable to write the model to Elasticsearch. This may be an addressing issue so look in the UR Engine's JSON in the `spark.es.nodes` value to make sure it is correct.
 - **Phase 6**: uses `curl` to send queries. These are recorded in and `actual...` file to be compare to `expected...` If the items returned in `actual` are the same as `expected` AND the ranking is identical, this means the results may be correct and only differ by the score. If the scores are different this may mean `expected` were created with a different version of Elasticsearch. This may or may not be a problem.
 - **Other Logs**: Failures may also have clues in the logs of MongoDB, Elasticsearch, and Spark.

## `sparkConf`

Other possible errors may occur due to an improper `sparkConf`. Settings here are specific to the Spark Job, not the services setup. These include:

 - **`master`** make sure this is the address of the port and host of the Spark master. "local" means use a Spark master inside the Harness process. If you are using an external Spark node the address should be something like `spark://spark-api:7077` or whatever address is shown in the Spark GUI.
 - **`spark.driver.memory`** this much memory must be available wherever Harness is running since the Spark Driver is inside the Harness process. This must be large enough to create a hashmap of all IDs in the data&mdash;so all user and item IDs. The smallest possible `spark.driver.memory` is `3g` and it goes up with the number of IDs.
 - **`spark.executor.memory`** this number * (number of executors) must be big enough to read in all data. Spark uses in-memory calculations but spreads them and the data across Executors.
 - **`spark.es.nodes`** should be the address that a Spark Executor can use to connect to Elasticsearch.