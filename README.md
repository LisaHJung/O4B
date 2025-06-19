# Demo architecture
<img width="1291" alt="image" src="https://github.com/user-attachments/assets/23e50dc5-e6fd-4f3f-bb6c-d7ed57f77081" />

## Objectives 
- Auto-instrument a Node.js app to generate traces and send traces to the OTel collector
- Configure the OTel collector to process traces and export the data to the Jaeger backend
- Use the Jaeger UI to visualize traces and to verify that the traces have been processed as intended

Note: Docker runs the OTel collector and Jaeger side-by-side with our app, so we can collect and view traces easily, without installing everything by hand.

## Roll the dice app
![Roll the dice mov](https://github.com/user-attachments/assets/79908390-adbc-4381-b81e-dffc67f0ea34)

## Run the app locally

**Project folder structure**

<img width="222" alt="image" src="https://github.com/user-attachments/assets/af683b58-f226-47c6-b94c-285479417cdf" />

- `app.js` runs the Roll the Dice app
- `instrumentation.js` auto-instruments the Roll the Dice app
- `OTel/otel-collector-config.yaml` configures the OTel collector
- `docker-compose.yaml` runs Jaeger and the OTel collector to collect and visualize traces from the Roll the Dice app

**Two project branches**
- `original-setup` branch shows the initial set up of the roll the dice app that send traces to the OTel collector.The collector uses the Resource attribute to add the "demo" service name and send traces to Jaeger.
- `post-processing` branch applies memory_limiter, Resource, Attributes, and Batch processors to the traces and sends the process traces to Jaeger. 

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

**Run the OTel collector and Jaeger using Docker**
```
//in the project directory
docker compose up --build 
```
**Refresh the Roll the Dice app page multiple times to send traces to the OTel collector**
In the terminal where you are running docker, you should be able to see traces being sent to the OTel collector in the OTel collector logs.
<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Verify that traces are being sent to Jaeger using its UI**
1. Go to the following URL: http://localhost:16686/

2. From the Jaeger UI, select the service named "demo" (orange box) then click on the "Find Traces" button. 
If you don't see it, try refreshing the roll the dice app page multiple times, then refreshing the Jaeger UI page.

3. Click on one of the traces (orange box)
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/89f53b47-ed71-4686-a7d7-420d85fab774" />

4. Click on the root span (Get/rolldice span)
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/8cf3bf48-ab8b-49e9-8d27-21c41aca9e0e" />

5. Expand tags and process section to view the metadata about traces collected from the app
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/caac086b-3409-44a3-a63d-055d75a0e78f" />
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cfd56101-3f5f-4dbf-8371-5a7cc1dfc693" />
<img width="1904" alt="image" src="https://github.com/user-attachments/assets/4a20c1e5-9ef0-446e-91e7-69f49e24def6" />

**Pay attention to info highlighted in orange as we will be using the OTel processors to remove them.**

Go to the post-processing branch using your terminal
```
//in the directory of the project
git checkout post-processing
``` 

**The OTel-collector configuration for this branch includes new processors (memory_limiter, resource, attributes, batch)**
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
**Run the OTel collector and Jaeger using Docker**
```
//in the project directory
docker compose down && docker compose up --build 
```
**Refresh the Roll the Dice app page multiple times to send traces to the OTel collector**
In the terminal where you are running docker, you should be able to see traces being sent to the OTel collector in the OTel collector logs.
<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Verify that traces are being sent to Jaeger using its UI**
1. Go back to the [Jaeger UI](http://localhost:16686/) 

2. From the Jaeger UI, select the service named "demo" (orange box) then click on the "Find Traces" button. 

3. Click on one of the traces (orange box)
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/89f53b47-ed71-4686-a7d7-420d85fab774" />

4. Click on the root span (Get/rolldice span)
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/8cf3bf48-ab8b-49e9-8d27-21c41aca9e0e" />

5. Expand tags and process section to view the metadata about traces collected from the app
<img width="1918" alt="image" src="https://github.com/user-attachments/assets/b518d096-3e9e-4d3f-a09c-0e51b1424fc2" />

6. Verify whether our traces have been processed correctly with the new OTel collector configuration

Original set up:
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cfd56101-3f5f-4dbf-8371-5a7cc1dfc693" />

Post-processing:
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/95a091dd-c6a6-4604-b81d-a95a06bac581" />

Original set up:
<img width="1904" alt="image" src="https://github.com/user-attachments/assets/4a20c1e5-9ef0-446e-91e7-69f49e24def6" />

Post-processing:
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/871015c3-ac6a-4640-a033-82a4a967ebbe" />
