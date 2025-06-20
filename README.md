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
- [`original-setup`](https://github.com/LisaHJung/O4B/tree/original-setup) branch shows the initial set up of the Roll the Dice app that send traces to the OTel collector. The collector uses the `Resource` attribute to add the "demo" service name to the traces and sends the traces to Jaeger for storage and visualization. 
- [`post-processing`](https://github.com/LisaHJung/O4B/tree/post-processing) branch includes the new OTel collector configuration. The following processors (`memory_limiter`, `Resource`, `Attributes`, and `Batch`) are applied to the traces before being sent to Jaeger. 

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
1. Go to the following URL: http://localhost:16686/

2. From the Jaeger UI, select the service named "demo" (orange box) then click on the "Find Traces" button. 
If you don't see it, try refreshing the roll the dice app page multiple times, then refreshing the Jaeger UI page.

3. Click on one of the traces (orange box)
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/89f53b47-ed71-4686-a7d7-420d85fab774" />

4. Click on the root span (Get/rolldice span)
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/b5e437b8-2cda-4760-93b6-0cc8dcbdba7b" />

5. Expand the tags and process sections to view the metadata about traces collected from the app
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/c62045a1-dd8e-45e0-8014-c8a8388dd87b" />

**Pay attention to info highlighted in orange as we will be using the OTel processors to remove them in the next section.**

<img width="1920" alt="image" src="https://github.com/user-attachments/assets/cfd56101-3f5f-4dbf-8371-5a7cc1dfc693" />
<img width="1904" alt="image" src="https://github.com/user-attachments/assets/4a20c1e5-9ef0-446e-91e7-69f49e24def6" />

## Add more processors to the OTel configuration to enrich and process the data

**[Original OTel collector configuration](https://github.com/LisaHJung/O4B/blob/original-setup/otel/otel-collector-config.yaml)**

In the previous section, the following OTel collector configuration was used to:
- collect traces from the app
- apply the `Resource` processor to add the service name "demo" to the traces
- export the traces to the Jaeger backend
  
<img width="661" alt="image" src="https://github.com/user-attachments/assets/66d6a4e6-e1cc-4190-9b62-2bff6be3157d" />

**[New OTel collector configuration](https://github.com/LisaHJung/O4B/blob/post-processing/otel/otel-collector-config.yaml)**

In the new configuration, we add the following processors to process the traces
- memory-delimiter
- resource
- attributes
- batch

The processed traces are sent to the Jaeger backend for storage and visualized using its UI. 

<img width="645" alt="image" src="https://github.com/user-attachments/assets/2938a98f-ff30-4c30-a9e9-da6563fb8527" />
<img width="645" alt="image" src="https://github.com/user-attachments/assets/58a54fba-7ddd-45ab-be55-2ca9f3055cd7" />

Switch to the post-processing branch using your terminal
```
//in the directory of the project
git checkout post-processing
``` 
**Run the OTel collector and Jaeger using Docker**
```
//in the project directory
docker compose down && docker compose up --build 
```
**Refresh the Roll the Dice app page multiple times to send traces to the OTel collector**
Take a look at the terminal that is running Docker which should be displaying the OTel collector logs. You should be able to see the traces being sent to the OTel collector.
<img width="1040" alt="image" src="https://github.com/user-attachments/assets/36081b69-8d28-4e16-9afa-86957759fc90" />

**Verify that traces are being sent to Jaeger using its UI**
1. Go back to the [Jaeger UI](http://localhost:16686/) 

2. From the Jaeger UI, select the service named "demo" (orange box) then click on the "Find Traces" button. 

3. Click on one of the traces (orange box)
<img width="1919" alt="image" src="https://github.com/user-attachments/assets/89f53b47-ed71-4686-a7d7-420d85fab774" />

4. Click on the root span (Get/rolldice)
<img width="1920" alt="image" src="https://github.com/user-attachments/assets/8cf3bf48-ab8b-49e9-8d27-21c41aca9e0e" />

5. Expand tags and process sections to view the metadata about traces collected from the app
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
