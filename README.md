# FoundationDB Built-in Testing

## Test Nodes

FoundationDB nodes must be specified as a `test` process so that the tests can be executed.  Using the operator, this means updating your FoundationDBCluster resource to specify the `processCounts` for the number of pods you want to run as `test` nodes.

```
kind: FoundationDBCluster
    ...
spec:
    ...
  processCounts:
    test: 2
        ...
```

## Test File To Run

The tests are defined in text files.  Example tests can be found in the [FoundationDB GitHub Repository](https://github.com/apple/foundationdb/tree/main/tests).  There is an example already defined in the ConfigMap definition in `test-cm.yaml`.

### Create ConfigMap with Test Configuration

The test configuration file needs to be available in the `test` pods to be executed.  You can update the `test-cm.yaml` file to include your test configuration and then create the ConfigMap:

```
oc apply -f test-cm.yaml
```

### Mount as ConfigMap

```
kind: FoundationDBCluster
    ...
spec:
    ...
  processes:
    test:
      podTemplate:
        spec:
          volumes:
            - name: test-conf
              configMap:
                name: test-cm
            - name: test-data
              emptyDir: {}
          containers:
          - name: foundationdb
                ...
            volumeMounts:
            - name: test-conf
              mountPath: /data/tst
            - name: test-data
              mountPath: /data/tst
        ...
```

## Run Test File

Tests are executed using the fdbserver binary available in the FoundationDB `test` pods.  Using the `test` profile will run the test on only the `test` node you started it on.  Using the `multitest` profile will run the test on all the `test` nodes including the node you started it on.

### Run on Single Test Node

```
fdbserver -f /data/tst/test.txt -r test > results.txt & disown
```

### Run on All Test Nodes

```
fdbserver -f /data/tst/test.txt -r multitest > results.txt & disown
```

# Aggregate Results

TODO...