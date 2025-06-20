# Demo architecture
<img width="1291" alt="image" src="https://github.com/user-attachments/assets/23e50dc5-e6fd-4f3f-bb6c-d7ed57f77081" />

## Objectives 
- Auto-instrument a Node.js app to generate traces and send traces to the OTel collector
- Configure the OTel collector to process traces and export the data to the Jaeger backend
- Use the Jaeger UI to visualize traces and to verify that the traces have been processed as intended

Note:

Docker runs the OTel collector and Jaeger side-by-side with our app, so we can collect and view traces easily, without installing everything by hand.

## Roll the dice app
![Roll the dice mov](https://github.com/user-attachments/assets/79908390-adbc-4381-b81e-dffc67f0ea34)

## Run the app locally

**Project folder structure**

<img width="222" alt="image" src="https://github.com/user-attachments/assets/af683b58-f226-47c6-b94c-285479417cdf" />

- `app.js` runs the Roll the Dice app
- `instrumentation.js` auto-instruments the Roll the Dice app
- `otel/otel-collector-config.yaml` configures the OTel collector
- `docker-compose.yaml` runs Jaeger and the OTel collector to collect, store, and visualize traces from the Roll the Dice app

**Two project branches**
- [`original-setup`](https://github.com/LisaHJung/O4B/tree/original-setup) branch includes the initial set up of the Roll the Dice app, the OTel collector, and Docker that runs both the OTel collector and Jaeger. 
  - This project is set up to generate traces and send traces to the OTel collector.
  - The OTel collector is configured receive and export the traces to Jaeger for storage and visualization.
- [`post-processing`](https://github.com/LisaHJung/O4B/tree/post-processing) branch includes the same set up except that:
  - additional processors were added to the OTel collector configuration to further process the traces before they are sent to Jaeger

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
**Refresh the Roll the Dice app page multiple times to send traces to the OTel collector**

Take a look at the terminal that is running Docker which should be displaying the OTel collector logs. You should be able to see the traces being sent to the OTel collector.
<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Verify that traces are being sent to Jaeger using its UI**
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

**Pay attention to info highlighted in orange as we will be using the OTel processors to remove them in the next section.**

**Tags section**

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cfd56101-3f5f-4dbf-8371-5a7cc1dfc693" />

**Process section**

<img width="1904" alt="image" src="https://github.com/user-attachments/assets/4a20c1e5-9ef0-446e-91e7-69f49e24def6" />

## Add more processors to the OTel configuration to enrich and process the data

**[Original OTel collector configuration](https://github.com/LisaHJung/O4B/blob/original-setup/otel/otel-collector-config.yaml)**

In the original configuration, the OTel collector was configured to use the:
- `otlp` receiver to receive traces from the app
- `resource` processor to add the "demo" service name to the traces
- `debug` exporter to print the traces being received by the collector
- `otlp/jaeger` exporter to send the traces to Jaeger for storage and visualization. 

<img width="661" alt="image" src="https://github.com/user-attachments/assets/66d6a4e6-e1cc-4190-9b62-2bff6be3157d" />

**[New OTel collector configuration](https://github.com/LisaHJung/O4B/blob/post-processing/otel/otel-collector-config.yaml)**

In the new configuration, we add the following processors to the original configuration to process the traces even further
- `memory-delimiter` processor prevents the collector from using too much memory by slowing down or dropping data when needed.
- `resource` processor modifies resource attributes
- `attributes` processor modifies or removes/adds attributes on spans
- `batch` processor batches spans to reduce network overhead

Both `memory-delimiter` and `batch` processors are recommended in all production set up.

<img width="645" alt="image" src="https://github.com/user-attachments/assets/2938a98f-ff30-4c30-a9e9-da6563fb8527" />
<img width="645" alt="image" src="https://github.com/user-attachments/assets/58a54fba-7ddd-45ab-be55-2ca9f3055cd7" />

**Switch to the [`post-processing`](https://github.com/LisaHJung/O4B/tree/post-processing) branch using your terminal**
```
//in the directory of the project
git checkout post-processing
``` 
**Run the OTel collector and Jaeger using Docker**
```
//in the project directory
docker compose down && docker compose up --build 
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
