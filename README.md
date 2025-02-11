# Application Setup and Configuration with Log Aggregation

## Introduction
This document outlines the setup and configuration process for a MERN stack application integrated with Grafana, Loki, and Promtail for log aggregation and visualization. It provides detailed steps for setting up the front-end, back-end, log collection, and dashboard creation.

## Features
- MERN stack application with customizable ports for front-end and back-end.
- Integrated Grafana dashboard for monitoring and visualizing logs.
- Log aggregation using Loki and Promtail for efficient storage and querying.
- Scalable and configurable architecture for real-time log insights.

## Prerequisites
Before proceeding, ensure the following prerequisites are met:
- A server/VM (e.g., AWS EC2) with Ubuntu or any Linux-based OS.
- Node.js and npm installed.
- Grafana installed.
- Loki and Promtail installed.
- Basic knowledge of system ports and application configurations.

## Installation

## 1. Clone the Repository

```bash
git clone https://github.com/your-repository/application.git
cd application
```

### Connecting the Backend Code to MongoDB and Hosting the Application

### Steps to Connect the Backend to MongoDB

1. **Navigate to the Backend Directory**:
   After cloning the Travel Memory application repository on your EC2 instance, navigate to the backend directory:
   ```bash
   cd /home/ubuntu/TravelMemory/
   cd Backend
   sudo touch .env # Created an environment file to store sensitive information such as the MongoDB URI and port number:
   nano .env
   #Pasted the below lines inside .env file
   #MONGO_URI='mongodb+srv://prasannak:*********@cluster0.7powl.mongodb.net/Travel-Memory-DB'
   #PORT=3001
   sudo apt install npm # Installed NPM and Dependencies and installed the necessary Node.js dependencies:
   npm install
   node index.js # Start the backend server
   ```
   Output on the EC2 inside : Server started at http://localhost:3001
   Now, went back to AWS console, when to my instance, went to "Security", went inside my security, clicked on edit inbound rules and added the below port details:
   ```bash
   custom-TCP TCP 3001 Custom 0.0.0.0/0
   ```
     
   Copied the public IP Address of the EC2 instance, went to the browser and pasted the below url :
   ```bash
   http://34.223.88.56:3001/
   ```
 


## 2. Establishing the Connection Between the Frontend and Backend Code Bases

1. Opened a new terminal on the same EC2 instance:
    ```bash
    cd /home/ubuntu/TravelMemory/
    cd Frontend
    cd src
    nano url.js
    ```

2. Added the following lines to `url.js`:
    ```javascript
    export const baseUrl = process.env.REACT_APP_BACKEND_URL || "http://34.223.88.56:3001/";
    ```
3. The public IP address of the EC2 instance where the backend application is hosted was pasted in the `url.js` file.
   
4. Started the frontend application:
    ```bash
    cd ..
    npm start
    ```
    

5. Updated Security Group Rules:
    - Went to the AWS console, navigated to the instance, then to "Security".
    - Edited the inbound rules and added a new rule for port 3000 and 3001 too.

6. Verified the setup by pasting the following IP address into the browser:
    ```plaintext
    http://34.223.88.56:3000/";
    ```
![Alt Text](/images/prom-tm.JPG)

## 3. Install and Configure Grafana

### Start Grafana
Start and enable the Grafana service using the following commands:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```
### Access Grafana
Grafana is accessible at:  
```bash
http://<server-ip>:3000
```

**Default credentials:**  
- **Username:** admin  
- **Password:** admin

### Change Grafana Port (Optional)
To change the port (e.g., to 5000):  

1. Open the `grafana.ini` configuration file:  
   ```bash
   sudo nano /etc/grafana/grafana.ini
   ```

2. Update the [server] section:
   ```bash
   [server]
   http_port = 5000
   ```

3. Restart Grafana:
   ```bash
   sudo systemctl restart grafana-server
   ```

  ![Alt Text](/images/prom-3-grafana-login-page.JPG)

  ![Alt Text](/images/prom-4-grafana-inside.JPG)

  
## 4. Install and Configure Loki

### Download and Set Up Loki
Run the following commands to download and set up Loki:

```bash
wget https://github.com/grafana/loki/releases/download/v2.8.0/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
chmod +x loki-linux-amd64
```

### Create Loki Configuration
Create a loki-config.yaml file with the following content:
```yaml
auth_enabled: false

server:
  http_listen_port: 3100

storage_config:
  boltdb_shipper:
    active_index_directory: /tmp/loki/index
    cache_location: /tmp/loki/cache
  filesystem:
    directory: /tmp/loki/chunks

scrape_configs:
  - job_name: 'mern-logs'
    static_configs:
      - targets: ['localhost']
        labels:
          job: 'mern-logs'
          __path__: /path/to/mern/logs/*.log

```

### Start Loki
Start Loki using the following command:
```bash
./loki-linux-amd64 -config.file=loki-config.yaml
```

## 5. Install Prometheus on Your EC2

### Download Prometheus
1. Go to the [Prometheus Downloads](https://prometheus.io/download/) page.
2. Copy the URL for the latest Prometheus version.
3. Download it on your Linux system:
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
   ```
4. Extract and Move the Files:
   ```bash
   tar -xvzf prometheus-2.47.0.linux-amd64.tar.gz
   mv prometheus-2.47.0.linux-amd64 prometheus
   cd prometheus
   ```
5. Run Prometheus
   ```bash
   ./prometheus --config.file=prometheus.yml
   ```
6. Set Up Node.js Backend for Prometheus Metrics:
   Navigate to Your Backend Folder
   Go to the directory where your Node.js backend application is located:
   ```bash
   cd ~/travel-memory-app/backend
   ```

7. Install Prometheus Client Library
   Run the following command to install the prom-client package:
   ```bash
   npm install prom-client
   ```

8. Modify the Backend Code to Expose Metrics
   Add the following code to your backend entry file (e.g., app.js or index.js):
   ```bash
   const express = require('express');
   const cors = require('cors');
   const promClient = require('prom-client');
   require('dotenv').config();

   const app = express();
   PORT = process.env.PORT;
   const conn = require('./conn');
   app.use(express.json());
   app.use(cors());

   // Prometheus metrics setup
   const collectDefaultMetrics = promClient.collectDefaultMetrics;
   collectDefaultMetrics(); // Collects default Node.js metrics

   // Create custom Prometheus metrics
   const httpRequestCounter = new promClient.Counter({
       name: 'http_requests_total',
       help: 'Total number of HTTP requests',
       labelNames: ['method', 'route', 'status']
   });

   const httpResponseTimeHistogram = new promClient.Histogram({
       name: 'http_response_time_seconds',
       help: 'Histogram of HTTP response times in seconds',
       labelNames: ['method', 'route', 'status'],
       buckets: [0.1, 0.5, 1, 1.5, 2, 5] // Define response time buckets
   });

   // Middleware to record metrics
   app.use((req, res, next) => {
       const start = Date.now();
       res.on('finish', () => {
           const responseTimeInSeconds = (Date.now() - start) / 1000;
           httpRequestCounter.inc({
               method: req.method,
               route: req.route ? req.route.path : req.path,
               status: res.statusCode
           });
           httpResponseTimeHistogram.observe(
               {
                   method: req.method,
                   route: req.route ? req.route.path : req.path,
                   status: res.statusCode
               },
               responseTimeInSeconds
           );
       });
       next();
   });

   // Routes
   const tripRoutes = require('./routes/trip.routes');
   app.use('/trip', tripRoutes); // http://localhost:3001/trip --> POST/GET/GET by ID

   app.get('/hello', (req, res) => {
       res.send('Hello World!');
   });

   // Expose Prometheus metrics endpoint
   app.get('/metrics', async (req, res) => {
       res.set('Content-Type', promClient.register.contentType);
       res.end(await promClient.register.metrics());
   });

   // Start server
   app.listen(PORT, () => {
       console.log(`Server started at http://localhost:${PORT}`);
   });

   ```

9. Save and Restart Your Backend Application:
   ```bash
   npm start
   ```
   The /metrics endpoint will now expose Prometheus metrics at http://localhost:3000/metrics.

 ![Alt Text](/images/prom-1.JPG)


 
 ![Alt Text](/images/prom-2-metrics.JPG)

## 6. Create a Grafana Dashboard
Add Loki as a Data Source
 1. Navigate to Configuration > Data Sources.
 2. Click Add Data Source and select Loki.
 3. Set the URL to your Loki instance (e.g., http://localhost:3100).
Build a Logs Dashboard
 1. Go to Dashboards > New Dashboard.
 2. Add a new Logs Panel.
 3. Query logs using labels defined in the Loki configuration (job="mern-logs").

 ![Alt Text](/images/prom-3-grafana-login-page.JPG)

 ![Alt Text](/images/prom-5-grafana-dashboard.JPG)

## 7. Testing the Setup
 1. Start the front-end and back-end applications.
 2. Check if logs are being generated in the specified directory.
 3. Verify that Loki and Promtail are pushing logs to Grafana.
 4. Open the Grafana dashboard to view and analyze the logs.

## 8. Troubleshooting
 1. Grafana not starting: Ensure no other service is using the configured port.
 2. Logs not appearing in Grafana: Verify Loki and Promtail configurations and log file paths.
  3. Loki service errors: Double-check the loki-config.yaml file for syntax issues.

## 9. Conclusion
This guide provides a step-by-step approach to setting up a MERN stack application with integrated log aggregation and visualization using Grafana and Loki. By following these steps, you can efficiently monitor your application's performance and troubleshoot issues in real time.
