# Corda Kubernetes deployment network for developers using Docker

In this example we are using the built-in Kubernetes that comes with Docker for Windows.  
If Kubernetes is not the wanted stack, just removing `--orchestrator=kubernetes` from the calls will deploy it in a docker stack instead.  

## Related
Network map, Identity Service and non-validating Notary: https://github.com/roastario/spring-boot-network-map  
Yo! CorDapp repository: https://github.com/corda/samples/tree/release-V3/yo-cordapp  
Corda: https://www.corda.net/  
Corda documentation: https://docs.corda.net/  

## Let us review the contents of this repository.

This folder contains a setup for distributing 3 [nodes](https://docs.corda.net/key-concepts-node.html)(party-a, party-b and party-c) in a network which includes an auto-accept Identity Service, a network map and a non-validating Notary.  
There is a script file that fetches the Corda Opensource 4.0 binaries (Source at: https://github.com/corda/corda), which is a required first step: *fetch_corda_jar*  
The folder contains a helpful script named *build-docker-nodes* which illustrates how to deploy this network in a Kubernetes environment.  

**NOTE!**   
If your git client retrieves .sh files with \r\n endings they have to be manually replaced with just \n endings or the script file cannot be executed.  
This applies to _all_ *.sh files in all sub-directories as well.  

## Usage  
Let us review the commands that are used to set up the Docker images and then to deploy the stack to Kubernetes.  

The following commands can be found in the *build-docker-nodes* script file as well, where it will run the commands in the correct order. But let us review the commands and what they do.  

**Remove any existing yo-app stacks.**  
```
docker stack rm yo-app --orchestrator=kubernetes
```

**Compiles the Docker images from the sub folders**  
```
docker build ./party-a/. -t party-a
docker build ./party-b/. -t party-b
docker build ./party-c/. -t party-c
```

**Deploy the stack**  
```
docker stack deploy yo-app --compose-file ./docker-compose.yml --orchestrator=kubernetes  
```

After it has been deployed, use this command to check that it is up and running:  
```
docker stack ps yo-app --orchestrator=kubernetes
```

From the above command you can also get the containers id and feed it into this command to view the output:  
```
docker service logs -f <CONTAINER>
```

The nodes also have SSH access to the [Node shell](https://docs.corda.net/shell.html), which allows you to execute any [flows](https://docs.corda.net/key-concepts-flows.html) directly on the nodes.    
Currently they can be accessed with username: **user1** and password: **test**, with the following command:  
```
ssh -o StrictHostKeyChecking=no user1@localhost -o UserKnownHostsFile=/dev/null -p 2221
```
**Note!**
In case you get an error that the connection was refused(`ssh: connect to host localhost port 2221: Connection refused`), please retry after about one minute, because until that point the Node is not yet fully operational.  
If you want to make sure it is ready before executing the ssh connect command, you can use the logs command mentioned above and wait until it is ready first.  

Please also note that depending on which port number you select, you will connect to *party-a(2221)*, *party-b(2222)* or *party-c(2223)*.  


Once in the Node Shell, you can initiate a Yo Flow by running the following command:  
```
flow start YoFlow target: [NODE_NAME], for example *PartyB*
```
Please note that the names of the parties are *PartyA*, *PartyB* and *PartyC*, these are the Nodes X500 names and should not be confused with the directory names which are all lower case.  

At this point you may consider logging in to another Node (party c) and sending a Yo to PartyB as well.  

In order to inspect if you have received a Yo from another Node, you should log in to party b using the ssh command from above just having the port number incremented. After that, you can execute the following command:  
```
run vaultQuery contractStateType: net.corda.yo.YoState
```

It should at this point list something similar to this:  
```
states:
- state:
    data: !<net.corda.yo.YoState>
      origin: "O=PartyA, L=London, C=GB"
      target: "O=PartyB, L=New York, C=US"
      yo: "Yo!"
    contract: "net.corda.yo.YoContract"
    notary: "O=Notary Service, L=London, C=GB"
    encumbrance: null
    constraint: !<net.corda.core.contracts.HashAttachmentConstraint>
      attachmentId: "3CBC4AB8BEC18532052AB568D1589DF4FF038160592F04E9F56EEAB79FEA70A1"
  ref:
    txhash: "01281C21ADF0ADCF50E8C47CE8855B5E7B0058B56619D168E76289814A854A1C"
    index: 0
statesMetadata:
- ref:
    txhash: "01281C21ADF0ADCF50E8C47CE8855B5E7B0058B56619D168E76289814A854A1C"
    index: 0
  contractStateClassName: "net.corda.yo.YoState"
  recordedTime: "2019-03-05T23:12:08.882Z"
  consumedTime: null
  status: "UNCONSUMED"
  notary: "O=Notary Service, L=London, C=GB"
  lockId: null
  lockUpdateTime: null
  relevancyStatus: "RELEVANT"
  constraintInfo:
    constraint:
      attachmentId: "3CBC4AB8BEC18532052AB568D1589DF4FF038160592F04E9F56EEAB79FEA70A1"
totalStatesAvailable: -1
stateTypes: "UNCONSUMED"
otherResults: []
```

At this point we have successfully executed a flow between multiple Nodes on the newly created test network!  

Please feel free and try other CorDapps at this point instead of the simple Yo-app.  
[Corda samples](https://github.com/corda/samples)
