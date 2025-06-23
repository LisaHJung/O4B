# Demo architecture
<img width="1907" alt="image" src="https://github.com/user-attachments/assets/644abade-998b-40d6-aef3-f249eefd0c77" />

## Objectives 
- Auto-instrument a Node.js app to generate traces and send traces to the OTel collector
- Configure the OTel Collector to process traces and export the data to the Jaeger backend
- Use the Jaeger UI to visualize traces and to verify that the data has been processed as intended

Note:

Docker runs the OTel Collector and Jaeger side-by-side with our app, so we can collect and view traces easily, without installing everything by hand.

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cb3b6fd1-19a3-4cb9-b92a-4497250f0a39" />

## Two project branches
1. [`original-setup`](https://github.com/LisaHJung/O4B/tree/original-setup)
- Instruments the Roll the Dice app and sends trace data to the OTel Collector.
- The Collector applies minimal processing (adds a service name) before forwarding the traces to Jaeger for storage and visualization.
2. [`post-processing`](https://github.com/LisaHJung/O4B/tree/post-processing) 
- Uses the same setup as original-setup, but adds additional processors to the OTel Collector configuration.
- These processors limit memory usage, enrich and clean up resource and attribute data, and batch traces for more efficient exporting.


## Run the demo locally

**Clone the project**
```
//directory of your choice
git clone https://github.com/LisaHJung/O4B.git
```
**Start the server**

Execute these commands in the terminal in the following order.
```
//in the project directory
npm install
npm start
```
**Verify the app is running**

In your browser, go to the following url: http://localhost:8080/rolldice

Refresh the page multiple times to see random numbers from 1-6 being generated on your screen. 
![Roll the dice mov](https://github.com/user-attachments/assets/79908390-adbc-4381-b81e-dffc67f0ea34)

**Run the OTel collector and Jaeger using Docker**
```
//in the project directory
docker compose up --build 
```
**Refresh the Roll the Dice app page multiple times to send traces to the OTel Collector**

Take a look at the terminal that is running Docker.

You will be able to see the logs of traces that are being sent to the Collector.

<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Verify that Jaeger is receiving traces from the OTel Collector**
1. Go the following url (http://localhost:16686/) to access the Jaeger UI. 

2. Click on the "Service" section (orange box) to view all the services that are sending traces to Jaeger.

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/5cfc0b94-990c-4e8f-9ebc-c58b73f850b2" />

In the OTel config, we set our service name to "demo".

Select the service "demo" then click on the "Find Traces" button (blue arrow).

If you don't see the service name "demo", try refreshing the Roll the Dice app page multiple times, then refreshing the Jaeger UI page.

3. Click on one of the traces (orange box)
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cf2e0018-c332-4c92-9ebf-9e88f86d8f6b" />

4. Click on the root span (Get/rolldice span)
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/b5e437b8-2cda-4760-93b6-0cc8dcbdba7b" />

5. Expand the tags and process sections to view the metadata about traces collected from the app
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/c62045a1-dd8e-45e0-8014-c8a8388dd87b" />

**Tags section**

The `Tags` section shows helpful details about what happened during a request, like which route was called, what method was used, or if any errors occurred. 

These details are shown as labels with values (like http.method = GET) that help you understand the behavior of your app.

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/930107d9-08b2-417a-b41d-5a2d80e54cc8" />

**Process section**

The `Process` section shows information about the app or service that created the trace. 

It includes things like the app’s name, the machine it ran on, and the command used to start it. This helps you see where the request came from and which service handled it — especially useful when you have multiple services or environments.

<img width="1919" alt="image" src="https://github.com/user-attachments/assets/999e8ed2-5c98-4b2f-9053-0bcb48cb4ba0" />

## Auto-instrumentation

In our setup, the following OTel packages have been installed:  
```
npm install @opentelemetry/sdk-node \
  @opentelemetry/api \
  @opentelemetry/auto-instrumentations-node \
  @opentelemetry/exporter-trace-otlp-grpc
```
**instrumentation.js**

```
const opentelemetry = require('@opentelemetry/sdk-node');

const {
  getNodeAutoInstrumentations,
} = require('@opentelemetry/auto-instrumentations-node');

const {
  OTLPTraceExporter,
} = require('@opentelemetry/exporter-trace-otlp-grpc');

const sdk = new opentelemetry.NodeSDK({
  traceExporter: new OTLPTraceExporter({
     url: 'http://localhost:4317', 
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();
```

`instrumentation.js` completes four tasks:

1. Import the required OTel packages needed for tracing:
```
const opentelemetry = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');
```

2. Set up the tracing system within our app by creating a new instance of the NodeSDK:

```
const sdk = new opentelemetry.NodeSDK({
  // configuration goes here
});
```

3. Configure the tracing system to automatically generate traces and export data to the local OTel Collector:

```
const sdk = new opentelemetry.NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: 'http://localhost:4317',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

```
4. Start the tracing system to begin recording traces and sending them to the collector:
```
sdk.start();
```
## OTel collector configuration
**otel/otel-collector-config.yaml**
```
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  resource:
    attributes:
      - key: service.name
        value: demo
        action: upsert

exporters:
  debug:
    verbosity: detailed

  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [resource]
      exporters: [debug, otlp/jaeger]
```
**Our OTel collector config has 4 components:**
1. Receivers
2. Processors
3. Exporters
4. Service pipelines

**`Receivers` tell the collector how to receive telemetry data**
```
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

```
- We configure the `OTLP` receiver to listen for incoming telemetry data in OTLP format on the specified ports: 4317 for gRPC and 4318 for HTTP.
- 
**Processors modify, filter, or enrich telemetry data before it gets exported**
```
processors:
  resource:
    attributes:
      - key: service.name
        value: demo
        action: upsert
```
- The `resource` processor updates or insers (`upsert`) the attribute `service.name = demo` in the telemetry data.
- Adding a service name is especially useful when you have multiple services sending telemetry to your observability backend.

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/5cfc0b94-990c-4e8f-9ebc-c58b73f850b2" />

**`Exporters` send data out to OTLP-compliant backends of your choice.**
```
exporters:
  debug:
    verbosity: detailed

  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true
```
- The `debug` exporter prints the telemetry data (traces, metrics) to the collector’s logs in a detailed way.
  - It’s useful for debugging or development to see what data is flowing through the collector.
  - 
 <img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />
 
- The `otlp/jaeger` exporter sends the telemetry data to a Jaeger backend at port 4317.


**`Service pipelines` configure how data flows inside the collector**
```
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [resource]
      exporters: [debug, otlp/jaeger]
```
- This defines a pipeline for traces.
- The traces sent from the app is received by the `otlp` receiver we defined in the configuration.
- The traces are processed by the `resource` processor, which adds the attribute `service.name = demo`.
- The `debug` exporter logs traces to the terminal where the collector is runnig.
- The `otlp/jaeger` exporter fowards the trace data to Jaeger. 

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cf2e0018-c332-4c92-9ebf-9e88f86d8f6b" />

In the `service.piplines` section, processors are applied sequentially in the order listed, so their order matters.

## Adding more processors to the OTel collector config 

**Switch to the [`post-processing`](https://github.com/LisaHJung/O4B/tree/post-processing) branch using your terminal**
```
//in the directory of the project
git checkout post-processing
``` 
**Run the OTel collector and Jaeger using Docker**
```
//in the project directory
CTRL + C
docker compose up --build 
```
The traces from the app should now be sent to the OTel collector with the new configuration that further processses the traces before they are sent to Jaeger. 

**Refresh the Roll the Dice app page multiple times to send traces to the OTel collector**

Take a look at the terminal that is running Docker which should be displaying the OTel collector logs. You should be able to see the traces being sent to the OTel collector.
<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Using the Jaeger UI, verify that traces were processed as intended**
1. Go back to the [Jaeger UI](http://localhost:16686/) 

2. Select the service named "demo" then click on the "Find Traces" button. 

3. Click on one of the traces 
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/0f5461bc-ef84-46c9-9e0f-0d2f4bda94d8" />

4. Click on the root span (Get/rolldice)
<img width="1915" alt="image" src="https://github.com/user-attachments/assets/d5946b07-126c-47c7-926b-d584ac247d73" />

5. Expand tags and process sections to view the metadata about traces collected from the app
<img width="1917" alt="image" src="https://github.com/user-attachments/assets/3acfaa72-d798-4f3d-bc02-d3e03ce6fedd" />

6. Verify whether our traces have been processed correctly with the new OTel collector configuration

**Traces from the collector with the original configuration:**
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cfd56101-3f5f-4dbf-8371-5a7cc1dfc693" />

**Traces from the collector with the new configuration:**
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/6b84ceab-d7fe-4039-ab86-2520105348ad" />

**Traces from the collector with the original configuration:**
<img width="1904" alt="image" src="https://github.com/user-attachments/assets/4a20c1e5-9ef0-446e-91e7-69f49e24def6" />

**Traces from the collector with the new configuration:**
<img width="1907" alt="image" src="https://github.com/user-attachments/assets/575f569d-364d-4a95-9dce-5f9979478614" />

### New OTel collector config

In the new configuration, we add the following processors to the original configuration to process the traces even further:
- `memory-delimiter` processor prevents the collector from using too much memory by slowing down or dropping data when needed.
- `resource` processor modifies resource attributes
- `attributes` processor modifies or removes/adds attributes on spans
- `batch` processor batches spans to reduce network overhead
  
Both `memory-delimiter` and `batch` processors are recommended in all production set up.

**otel/otel-collector-config.yaml**
```
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  memory_limiter:
    check_interval: 2s
    limit_mib: 512
    spike_limit_mib: 128

  resource:
    attributes:
      - key: service.name
        value: demo
        action: upsert
      - key: deployment.environment
        value: local
        action: insert
      - key: host.arch
        action: delete
      - key: host.id
        action: delete
      - key: process.command_args
        action: delete
      - key: process.executable.path
        action: delete

  attributes:
    actions:
      - key: http.user_agent
        action: delete
      - key: net.peer.ip
        action: delete
      - key: net.host.port
        action: delete
      - key: net.peer.port
        action: delete

  batch:
    timeout: 5s
    send_batch_size: 512

exporters:
  debug:
    verbosity: detailed

  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, resource, attributes, batch]
      exporters: [debug, otlp/jaeger]
```
**`memory-delimiter` processor prevents the collector from using too much memory by slowing down or dropping data when needed.**
```
processors:
  memory_limiter:
    check_interval: 2s
    limit_mib: 512
    spike_limit_mib: 128
```

**`resource` processor modifies resource attributes**
```
 resource:
    attributes:
      - key: service.name
        value: demo
        action: upsert
      - key: deployment.environment
        value: local
        action: insert
      - key: host.arch
        action: delete
      - key: host.id
        action: delete
      - key: process.command_args
        action: delete
      - key: process.executable.path
        action: delete
```
- `service.name` and `deployment.environment` resource attributes were added to the traces
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/56bf7028-ef30-4948-ac48-b469f69429bd" />
<img width="1918" alt="image" src="https://github.com/user-attachments/assets/fc6bc5fd-77e7-448e-8030-ab427058d913" />

The following resource attributes were deleted to remove sensitive or irrelevant data to reduce noise, improve privacy, and keep trace data focused.
  - host.arch
  - host.id
  - process.command_asrgs
  - process.executable.path
  
**`attributes` processor modifies resource attributes**

```
attributes:
  actions:
    - key: http.user_agent
      action: delete
    - key: net.peer.ip
      action: delete
    - key: net.host.port
      action: delete
    - key: net.peer.port
      action: delete
```

The following resource attributes were deleted to remove sensitive or personally identifiable information (PII). 
  - http.user_agent
  - net.peer.ip
  - net.host.port
  - net.peer.port

Deleting them enhances privacy and security compliance and reduces the size of trace payloads.  
