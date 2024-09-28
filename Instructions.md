## Overview

Building your first Cosmos-SDK blockchain with Spawn.
- Generating a new chain
- Creating a new module
- Adding custom logic
- Run locally
- Interacting with the network

### Generate a New Chain

```bah
spawn new hackmoschain --bech32=hack --denom=uhackmos --bin=hackmosd --org=rollchains

cd hackmoschain/

spawn module new ems

```

#### now add query method in query.proto

```bash
// ResolveName allows a user to resolve the name of an account.
  rpc GetEvent(QueryGetEventRequest) returns (QueryGetEventResponse) {
    option (google.api.http).get = "/ems/v1/name/{organizer}";
  }
  
  
  message QueryGetEventRequest {
  string organizer = 1;
}

message QueryGetEventResponse {
  string name = 1;
}

```

#### tx.proto

```bash
 rpc CreateEvent(MsgCreateEvent) returns (MsgCreateEventResponse);
  
  
  
  // MsgSetServiceName defines the structure for setting a name.
message MsgCreateEvent {
  option (cosmos.msg.v1.signer) = "organizer";

  string organizer = 1 [(cosmos_proto.scalar) = "cosmos.AddressString"];

  string name = 2;
}

// MsgSetServiceNameResponse is an empty reply.
message MsgCreateEventResponse {}

```

Protoc is the Protocol Buffers compiler. It reads .proto files (which define the structure of your data) and generates code based on those definitions. This generated code can include data structures and methods.
These .proto file templates will be converted into Golang source code for you to use. Build the Go source code using the command:

```bash
make proto-gen
```

#### Save Storage Structure
x/ems/keeper.go

You now need to set the data structure in the keeper to store the organizer and event name pair. Keeper's are where the data is stored for future use.

```bash
type Keeper struct {
	...
	EventMapping collections.Map[string, string]
}

func NewKeeper() Keeper {
  ...

  k := Keeper{
    ...
    EventMapping: collections.NewMap(sb, collections.NewPrefix(1), "event_mapping", collections.StringKey, collections.StringValue),
  }

}

```

#### Application Logic

Update the msg_server logic to set the event upon request from a user.

x/ems/keeper/msg_server.go

```bash
// CreateEvent implements types.MsgServer.
func (ms msgServer) CreateEvent(ctx context.Context, msg *types.MsgCreateEvent) (*types.MsgCreateEventResponse, error) {
    if err := ms.k.EventMapping.Set(ctx, msg.Organizer, msg.Name); err != nil {
        return nil, err
    }
    return &types.MsgCreateEventResponse{}, nil
}
```

and also for the query_server to retrieve the event.

```bash
// GetEvent implements types.QueryServer.
func (k Querier) GetEvent(goCtx context.Context, req *types.QueryGetEventRequest) (*types.QueryGetEventResponse, error) {
    v, err := k.Keeper.EventMapping.Get(goCtx, req.Organizer)
    if err != nil {
        return nil, err
    }

    return &types.QueryGetEventResponse{
        Name: v,
    }, nil
}
```


#### Command Line Client
Using the Cosmos-SDKs AutoCLI, you will easily set up the CLI client for transactions and queries.

- Query and Transaction
Update the autocli to allow someone to set and get the event.

```bash
func (am AppModule) AutoCLIOptions() *autocliv1.ModuleOptions {
    return &autocliv1.ModuleOptions{
        Query: &autocliv1.ServiceCommandDescriptor{
            Service: modulev1.Query_ServiceDesc.ServiceName,
            RpcCommandOptions: []*autocliv1.RpcCommandOptions{
                {
                    RpcMethod: "GetEvent",
                    Use:       "get [organizer]",
                    Short:     "Get event by organizer",
                    PositionalArgs: []*autocliv1.PositionalArgDescriptor{
                        {ProtoField: "organizer"},
                    },
                },
                {
                    RpcMethod: "Params",
                    Use:       "params",
                    Short:     "Query the current consensus parameters",
                },
            },
        },
        Tx: &autocliv1.ServiceCommandDescriptor{
            Service: modulev1.Msg_ServiceDesc.ServiceName,
            RpcCommandOptions: []*autocliv1.RpcCommandOptions{
                {
                    RpcMethod: "CreateEvent",
                    Use:       "create [name]",
                    Short:     "Create an event",
                    PositionalArgs: []*autocliv1.PositionalArgDescriptor{
                        {ProtoField: "name"},
                    },
                },
                {
                    RpcMethod: "UpdateParams",
                    Skip:      false, // set to true if authority gated
                },
            },
        },
    }
}
```

#### Launch The Network
Use the sh-testnet command (short for shell testnet) to quickly build your application, generate example wallet accounts, and start the local network on your machine.

# Run a quick shell testnet
```bash
make sh-testnet
```

Transactions to interact with the etm module
```bash
hackmosd tx ems -h

hackmosd tx ems create vitwit-event-1 --from=acc0

hackmosd q ems -h

hackmosd q ems get event1hj5fveer5cjtn4wd6wstzugjfdxzl0xp0a2w8n

```