# Demo architecture
<img width="1918" alt="image" src="https://github.com/user-attachments/assets/af320cc8-b7ac-4cca-8501-1b375f727df6" />

## Objectives 
- Auto-instrument a Node.js app with OTel SDK to generate traces and send traces to the OTel Collector.
- Configure the OTel Collector to receive, process, and export traces to the Jaeger backend.
- Use the Jaeger UI to visualize and verify that the traces have been correctly processed.

Note:
- Ensure that you have [Node.js](https://nodejs.org/en/download/) installed locally.
- Docker runs the OTel Collector and Jaeger side by side with our app, for easy set up and integration. 

<img width="1918" alt="image" src="https://github.com/user-attachments/assets/a398be28-3bbc-4895-b343-8cbc09fac606" />

## Two project branches
1. [`original-setup`](https://github.com/LisaHJung/O4B/tree/original-setup)
- Instruments the Roll the Dice app and sends trace data to the OTel Collector.
- The Collector forwards the traces to Jaeger for storage and visualization.
2. [`post-processing`](https://github.com/LisaHJung/O4B/tree/post-processing) 
- Uses the same setup as original-setup, but applies processors to traces. 
- These processors enrich and clean up resource and attribute data, and batch traces for more efficient exporting.

## Run the demo locally

**Clone the project**
```
//directory of your choice
git clone [update this]
```
**Start the server**

Execute the following commands.
```
//in the project directory
npm install
npm start
```
**Verify the app is running**

In your browser, go to the following URL: http://localhost:8080/rolldice

Refresh the page multiple times. This app will generate a random number from 1-6 just as if you were rolling a die. 
![Roll the dice mov](https://github.com/user-attachments/assets/514cbc4f-ac5c-4405-9d4f-61df9e4a301e)

**Using Docker, run the OTel Collector and Jaeger**
```
//project directory in a different terminal
docker compose up --build 
```
**Refresh the Roll the Dice app page multiple times to send traces to the OTel Collector**

![movie](https://github.com/user-attachments/assets/7ddea1df-87df-4817-88b1-1a35b1adfc2d)

Take a look at the terminal that is running Docker.

You will be able to see the logs of traces that are flowing through the Collector.

<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Verify that Jaeger is receiving traces from the OTel Collector**
1. Go to the following URL (http://localhost:16686/) to access the Jaeger UI. 

2. Click on the "Service" section (orange box) to view all the services that are sending traces to Jaeger.

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/5cfc0b94-990c-4e8f-9ebc-c58b73f850b2" />

In `package.json`, we set the environment variable "OTEL_SERVICE_NAME=demo".

Select the service "demo" then click on the "Find Traces" button (blue arrow).

If you don't see the service name "demo", try refreshing the Roll the Dice app page multiple times, and then refreshing the Jaeger UI page.

3. Click on one of the traces (orange box)
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cf2e0018-c332-4c92-9ebf-9e88f86d8f6b" />

4. Click on the root span (Get/rolldice span)
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/b5e437b8-2cda-4760-93b6-0cc8dcbdba7b" />

5. Expand the tags and process sections to view the metadata about traces collected from the app
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/826578a3-6f3c-4a33-acfb-f76997eeb9ff" />

**The `Tags` section shows details about what happened during a request.** 

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/930107d9-08b2-417a-b41d-5a2d80e54cc8" />

It consists of `span attributes` (i.e. routes, method, errors)  to help you understand the behavior of your app.

**The `Process` section shows information about the app or service that created the trace.**

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/b17572ff-1ac3-4559-a997-e0ab1e382923" />

It consists of `resource attributes` that present info about the machine it ran on, and the command used to start it. 

It helps you see where the request came from and which service handled it.

## Auto-instrumentation

In our setup, the following OTel packages have been installed:  
```
npm install @opentelemetry/sdk-node \
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
4. Start the tracing system to begin recording traces and sending them to the Collector:
```
sdk.start();
```
**IMPORTANT**

- The instrumentation setup and configuration must be run before your application code. One tool commonly used for this task is the –require flag.
- In a properly instrumented application, the servce name is set as an environment variable.
- In this project, the app start script in package.json sets the OTEL_SERVICE_NAME environment variable and uses the -require flag to to run the instrumentation setup and configuration before the application code. 
```
{
  "name": "latest",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "OTEL_SERVICE_NAME=demo node --require ./instrumentation.js app.js"
  },
  "keywords": [],
  "author": "",   
  "license": "ISC",
  "description": "",
  "dependencies": {
    "@opentelemetry/auto-instrumentations-node": "^0.60.1",
    "@opentelemetry/exporter-trace-otlp-grpc": "^0.202.0",
    "@opentelemetry/sdk-node": "^0.202.0",
    "express": "^5.1.0"
  }
}
```
## OTel Collector configuration
**otel/otel-collector-config.yaml**
```
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

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
      exporters: [debug, otlp/jaeger]
```
**Our OTel Collector configuration consists of 3 components:**
1. Receivers
2. Exporters
3. Service 

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
- We set up the OTLP receiver to accept incoming telemetry data on two ports so it can receive data from our app.
  

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
- The `debug` exporter prints the telemetry data to the Collector’s logs in a detailed way.
  - It displays what data is flowing through the Collector and is useful for debugging or development. 
  
 <img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />
 
- The `otlp/jaeger` exporter sends the telemetry data to a Jaeger backend at port 4317.


**The `Service` component configures how data flows inside the Collector**
```
service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [debug, otlp/jaeger]
```
- Our configuration defines a pipeline for traces.
- The traces sent from the app is received by the `otlp` receiver we defined in the configuration.
- The `debug` exporter logs traces to the terminal where the Collector is running.
- The `otlp/jaeger` exporter forwards the trace data to Jaeger. 

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cf2e0018-c332-4c92-9ebf-9e88f86d8f6b" />


## Process telemetry data within the OTel Collector 

**Switch to the [`post-processing`](https://github.com/LisaHJung/O4B/tree/post-processing) branch using your terminal**
```
//in the directory of the project
git checkout post-processing
``` 
**Stop and restart the OTel Collector and Jaeger**
```
//in the project directory
CTRL + C
docker compose up --build 
```
**Refresh the Roll the Dice app page multiple times to send the traces to the newly configured OTel Collector**

<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Using the Jaeger UI, examine the traces to verify that they have been processed correctly.**
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/72ea707d-321d-42b7-89a3-4b11a4114ccd" />
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/e6dfa298-5721-49fb-abaa-8b0e89af651c" />

### New OTel collector configuration
**Processors modify, filter, or enrich telemetry data before it gets exported**
We added 3  processors to the original config:
- `resource` 
- `attributes` 
- `batch` 

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
      - key: deployment.environment.name
        value: local
        action: insert
      - key: host.arch
        action: delete
      - key: host.id
        action: delete
      - key: host.name
        action: delete  
      - key: process.command
        action: delete
      - key: process.command_args
        action: delete
      - key: process.executable.path
        action: delete
      - key: process.owner
        action: delete
      - key: process.pid
        action: delete

  attributes:
    actions:
      - key: http.user_agent
        action: delete
      - key: net.host.ip
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
      processors: [resource, attributes, batch]
      exporters: [debug, otlp/jaeger]
```

**The `resource` processor modifies metadata about the service or host.**
```
 resource:
    attributes:
      - key: deployment.environment.name
        value: local
        action: insert
      - key: host.arch
        action: delete
      - key: host.id
        action: delete
      - key: host.name
        action: delete  
      - key: process.command
        action: delete
      - key: process.command_args
        action: delete
      - key: process.executable.path
        action: delete
      - key: process.owner
        action: delete
      - key: process.pid
        action: delete

```
- `deployment.environment.name` resource attributes were added to the traces.

<img width="1918" alt="image" src="https://github.com/user-attachments/assets/fc6bc5fd-77e7-448e-8030-ab427058d913" />

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
      - key: host.name
        action: delete  
      - key: process.command
        action: delete
      - key: process.command_args
        action: delete
      - key: process.executable.path
        action: delete
      - key: process.owner
        action: delete
      - key: process.pid
        action: delete

```
The following resource attributes were deleted to remove sensitive or irrelevant data.
This is done to reduce noise, improve privacy, and keep trace data focused.
  - host.arch
  - host.id
  - host.name
  - process.command
  - process.command_asrgs
  - process.executable.path
  - process.owner
  - process.pid

**Traces from the original OTel Collector configuration:**
<img width="1905" alt="image" src="https://github.com/user-attachments/assets/d54248b6-2dd8-4ff2-a4d5-515cdd8ad7fb" />

**Traces from the new OTel Collector configuration:**
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/fac109ca-1ff3-4c8d-9df2-054c7bbf4fd5" />

**The `attributes` processor modifies, adds, or removes attributes on spans.**

```
attributes:
    actions:
      - key: http.user_agent
        action: delete
      - key: net.host.ip
        action: delete
      - key: net.host.port
        action: delete
      - key: net.peer.ip
        action: delete
      - key: net.peer.port
        action: delete
```

The following resource attributes were deleted to remove sensitive or personally identifiable information (PII). 
  - http.user_agent
  - net.peer.ip
  - net.host.port
  - net.peer.ip
  - net.peer.port

Deleting them enhances privacy and security compliance and reduces the size of trace payloads.  

**Traces from the original OTel Collector configuration:**
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/c138b6d7-bce3-4691-b992-693a90bbeebe" />

**Traces from the new OTel Collector configuration:**
<img width="1910" alt="image" src="https://github.com/user-attachments/assets/4edc18b8-4008-4331-a5e8-3b0fe1a2706f" />

**The `batch` processor groups telemetry data into batches before exporting.**
```
batch:
    timeout: 5s
    send_batch_size: 512

```
It improves performance and reduces overhead. This processor is recommended to be added to the collector configuration as a best practice. 

**The Service component was updated to include the processors that have been added.**
```
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, resource, attributes, batch]
      exporters: [debug, otlp/jaeger]
```

**Note:**
In the `service` section, the processors are applied sequentially in the order listed, so pay attention to the order! 

The `batch` processor should be listed *last* to group the data into batches before exporting. 


## Resources
- [OTel documentation](https://opentelemetry.io/docs/)
  - Ask AI (upper right corner of the OTel documentation)
  - [Language APIs and SDKs](https://opentelemetry.io/docs/languages/)
  - [Instrumentation](https://opentelemetry.io/docs/concepts/instrumentation/)
  - [OTel Collector](https://opentelemetry.io/docs/collector/)
    - [List of OTel Collector processors](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor) 
- [OTel YouTube channel](https://www.youtube.com/@otel-official)
  - [OTel for Beginners series - The JavaScript Journey](https://youtu.be/iEEIabOha8U?feature=shared)
    - Stay tuned for videos on this talk + working with other telemetry types. 

## QR code for project repo:
![image](https://github.com/user-attachments/assets/0ac2cc04-77aa-492e-8dfd-f582151df1ec)

## QR code for lightning talk 
![image](https://image-charts.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl=https://sched.co/25vDo)
