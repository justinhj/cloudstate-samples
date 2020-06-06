
## Layout
* `myservice.proto` - This is the gRPC interface that is exposed to the rest of the world. The user function doesn't implement this directly, it passes it to the Akka backend, that implements it, and then proxies all requests to the user function through an event sourcing specific protocol. Note the use of the `cloudstate.entity_key` field option extension, this is used to indicate which field(s) form the entity key, which the Akka backend will use to identify entities and shard them.
* `domain.proto` - These are the protobuf message definitions for the domain events and state. They are used to serialize and deserialize events stored in the journal, as well as being used to store the current state which gets serialized and deserialized as snapshots when snapshotting is used.
* `myservice.js` - This is the JavaScript code for implementing the service entity user function. It defines handlers for events and commands. It uses the cloudstate-event-sourcing Node.js module to implement the event sourcing protocol.
* `deploy` directory that contains the deployment YAML files.

### Storage Setup
* Modify `deploy/postgres-store.yaml`
    * Change the `name` to be unique amongst your services.
    * eg: `myservice-postgres`
* Modify `my-service.yaml` to match
    * Change `spec|storeConfig|statefulStore|name` to match the name used above.

## Your Code
You want to make changes to the following files:
* Edit both the `domain.proto` and the `myservice.proto`, renaming those files as you see fit.
* Once your model, command and events are defined as you like, we need to know how to implement the service.
* Edit the `index.js` file, again renaming the file and contents to your liking.
   * Follow the comments in these files as a guide to help you edit.

## Building
```
nvm install
nvm use
npm install
npm run prestart
```

This will create `user-function.desc` which describes your stateful function to Cloudstate.
```
$ docker build . -t <my-registry>/my-service:latest
```

Push the Docker image to the registry:
```
$ docker push <my-registry>/my-service:latest
```

Deploy the image by changing into the deploy folder and editing the `my-service.yaml` to point to the Docker image that you just pushed:
```
$ cd ../deploy

$ cat my-service.yaml
apiVersion: cloudstate.io/v1alpha1
kind: StatefulService
metadata:
  name: my-service
spec:
  containers:
  - image: coreyauger/my-service:latest    # <-- Change this to your image
    name: my-service
```

## Deploy

Deploy the stateful store for your project to use into your project's Kubernetes namespace:
```
$ kubectl apply -f postgres-store.yaml -n <project-name>
statefulstore.cloudstate.io/my-postgres created
````

Deploy the stateful service for your project into your project's Kubernetes namespace:
```
$ kubectl apply -f my-service.yaml -n <project-name>
statefulservice.cloudstate.io/my-service created
````

### Verify they are ready
Check that the stateful store and service are in the "Ready" state:

```
$ kubectl get statefulstore -n <project-name>
NAME          AGE   STATUS
my-postgres   25m   Ready

$ kubectl get statefulservices -n <project-name>
NAME         AGE   REPLICAS   STATUS
my-service   25m   1          Ready
```

To redeploy a new image to the cluster you must delete and then redeploy using the YAML file. For example, if we updated the my-service Docker image we would do the following:
````
$ kubectl delete statefulservice my-service -n <project-name>
statefulservice.cloudstate.io "my-service" deleted

$ kubectl apply -f my-service.yaml -n <project-name>
statefulservice.cloudstate.io/my-service created
````

## Routes
Public routes can be used through gRPC and gRPC-Web calls.  These exist in the `routes.yaml` file.

```
$ cat routes.yaml
apiVersion: cloudstate.io/v1alpha1
kind: Route
metadata:
  name: my-routes
spec:
  http:
  - name: my-routes
    match:
    - uri:
        prefix: /
    route:
      service: my-service
```

Add these routes by performing:
```
$ kubectl apply -f routes.yaml -n <project-name>
```

The web URL that will resolve the above route to the default route:
`https://<project-name>.us-east1.apps.lbcs.io/`

NOTE: utilities like `grpcurl` use service reflection on the default route `/`.  What this means is that you can only use `grpcurl` with the service on the default route. Using `grpcurl` when the service supports reflection on its default route might looks as follows.

List available services:

```
$ grpcurl kw-alpha4.us-east1.apps.lbcs.io:443 list
com.example.myservice.MyService
```

Describe service to retrieve the schema that the service is using:

```
$ grpcurl <project-name>.us-east1.apps.lbcs.io:443 describe com.example.myservice.MyService
com.example.myservice.MyService is a service:
service MyService {
  rpc AddItem ( .com.example.myservice.MyAddItem ) returns ( .google.protobuf.Empty ) {
    option (.google.api.http) = { post:"/state/{user_id}/items/add" body:"*"  };
  }
  rpc GetState ( .com.example.myservice.MyGetState ) returns ( .com.example.myservice.MyState ) {
    option (.google.api.http) = { get:"/state/{user_id}" additional_bindings:<get:"/state/{user_id}/items" response_body:"items" >  };
  }
  rpc RemoveItem ( .com.example.myservice.MyRemoveItem ) returns ( .google.protobuf.Empty ) {
    option (.google.api.http) = { post:"/state/{user_id}/items/{id}/remove"  };
  }
}
```

## Testing
You can now use `curl` in combination with the `option (google.api.http)` that you defined in your `myservice.proto`.  For more information on how json is encoded to protobuf see: [https://cloud.google.com/endpoints/docs/grpc/transcoding](https://cloud.google.com/endpoints/docs/grpc/transcoding).

### Adding an item
```
$ curl -v -XPOST -H "Content-Type:application/json" https://<PROJECT_NAME>.us-east1.apps.lbcs.io/state/{user_id}/items/add -d '{"user_id":"username", "id":"test", "name":"test", quantity: 1}'
```

### Listing added items
```
$ curl -v -H "Content-Type:application/json" https://<PROJECT_NAME>.us-east1.apps.lbcs.io/state/{user_id}/items
```

### Removing an item
```
$ curl -v -XPOST -H "Content-Type:application/json" https://<PROJECT_NAME>.us-east1.apps.lbcs.io/state/{user_id}/items/{id}/remove
```

## Maintenance notes

### License
The license is Apache 2.0, see [LICENSE-2.0.txt](LICENSE-2.0.txt).

### Maintained by
__This project is NOT supported under the Lightbend subscription.__

If you find any issues with these instructions, please report them [here](https://github.com/lightbend/cloudstate-samples/pull/link_to_issue_tracker).

Feel free to ping the maintainers above for code review or discussions. Pull requests are very welcome–thanks in advance!


### Disclaimer

[DISCLAIMER.txt](../DISCLAIMER.txt)
