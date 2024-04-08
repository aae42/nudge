# nudge

Not as forceful as a push.

## Design

The `Node` is a Chef node running `nudge` as a service.
The `Message Queue` is a server running [nats message queue](https://nats.io/).
`Pipeline` is a CI job running `nudge` to interact with the queue.

```mermaid
sequenceDiagram
  loop
    Node->>Message Queue: Subscribe to `<service>.<policy-name>.<policy-group>.run`
    Pipeline->>Message Queue: Post a message to `<service>.<policy-name>.<policy-group>.run`, "run!"
    Message Queue->>Node: Node reads the message from `<service>.<policy-name>.<policy-group>.run`
    Node->>Message Queue: Node posts a message to `<service>.<policy-name>.<policy-group>.status`, "{"server":<fqdn>, "running": true}"
    Note over Node: runs `cinc-client`
    Message Queue->>Pipeline: Reads messages from `<service>.<policy-name>.<policy-group>.status` to output running clients
    Node->>Message Queue: Node posts a message to `<service>.<policy-name>.<policy-group>.status`, "{"server":<fqdn>, "cinc-client": "successful"}" or "unsuccessful" at end of cinc-client run
    Node->>Message Queue: Node posts test results to `<service>.<policy-name>.<policy-group>.status`, "{"server":<fqdn>, test_results: <inspec json>}"
    Message Queue->>Pipeline: Reads messages from `<service>.<policy-name>.<policy-group>.status`
    Note over Pipeline:  show overall success or a breakdown of failures
  end
```

## NATS Subject setup

- service
  - \*.\*.run
    - pipeline (environment split?) write
  - \*.\*.status
    - pipeline read
  - policy-name.policy-group.run
    - nodes read
  - policy-name.policy-group.status
    - nodes write
