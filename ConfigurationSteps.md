
## Travel Memory Application Deployment Guide

### Objective

The goal is to host the **Travel Memory Application** (MERN stack) on a custom domain using an **Application Load Balancer (ALB)** and **Nginx** server with **AWS services**. The application's GitHub repository is available at `https://github.com/jatinggg/TravelMemory.git`.

-----

## Configuration Steps

### Backend Configuration

1.  **Start an EC2 Instance and Connect**.
2.  **Clone the Git repository**:
    ```bash
    sudo apt git clone https://github.com/jatinggg/TravelMemory.git
    ```
3.  **Configure the environment file**:
      * Navigate to the backend directory: `cd TravelMemory/backend`.
      * Open the environment file for editing: `sudo nano .env`].
      * Configure the **URI for the MongoDB database** and set the port to **3000**.
4.  **Install and Start Nginx**:
    ```bash
    sudo apt install nginx -y
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```
      * Verify the status: `sudo systemctl status nginx`.
5.  **Install Node.js and npm**:
    ```bash
    sudo apt install nodejs
    sudo apt install npm
    ```
6.  **Install Dependencies**:
      * Run `sudo npm install` to install React dependencies in the backend directory.
7.  **Set up Nginx Reverse Proxy**:
      * Open the Nginx configuration file: `sudo nano /etc/nginx/sites-available/default`.
      * Paste the following configuration to set up a reverse proxy on port **80**:
        ```nginx
        server {
            listen 80;
            location / {
                proxy_pass http://<ec2-public-ip>:3000;  # Change this to your backend app URL
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
        ```
8.  **Test and Reload Nginx**:
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```
9.  **Update EC2 Security Group**:
      * Allow **TCP at port 80** for `0.0.0.0/0`.
10. **Run the Backend Server**:
      * Navigate to the backend directory: `cd TravelMemory/backend`.
      * Run: `sudo node index.js`. The server will start at `http://localhost:3000`.

#### Creating a Second Backend Server for Load Balancing

1.  **Create a Machine Image** of the first instance.
2.  **Launch a new instance** using this image.
3.  **Configure Nginx on the second instance**:
      * Update `/etc/nginx/sites-available/default` and replace the IP address with the **public IP of the second backend server**.
4.  **Test and Reload Nginx**:
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```
5.  **Run the Backend Server**:
      * Navigate to the backend directory: `cd TravelMemory/backend`.
      * Run: `sudo node index.js`.

-----

### Frontend Configuration

1.  **Start an EC2 Instance and Connect**.
2.  **Clone the Git repository**:
    ```bash
    sudo apt git clone https://github.com/jatinggg/TravelMemory.git
    ```
3.  **Install and Start Nginx**:
    ```bash
    sudo apt install nginx -y
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```
4.  **Install npm**:
    ```bash
    sudo apt install npm
    ```
5.  **Configure Backend URL**:
      * Navigate to the frontend source: `cd TravelMemory/frontend/src`.
      * Edit `url.js`: `sudo nano url.js`.
      * Configure the backend URL: `"http://<your-backend-public-ip:3000>"`.
6.  **Install Dependencies**:
      * Navigate to the frontend directory: `cd TravelMemory/frontend`.
      * Run: `sudo npm install`.
7.  **Set up Nginx Reverse Proxy for Frontend**:
      * Open the configuration file: `/etc/nginx/sites-available/default`.
      * Under `proxy_pass`, add your **frontend instance's public IP address**.
      * Ensure the instance's inbound rules allow **TCP traffic to port 80** from `0.0.0.0/0`.
8.  **Test and Reload Nginx**:
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```
9.  **Run the Frontend Application**:
      * Navigate to the frontend directory: `cd TravelMemory/frontend`.
      * Run: `sudo npm start`.
10. **Verify Application**:
      * Open `http://<-frontend-public-ip->` in a web browser to check the application is working.

#### Creating a Second Frontend Server for Load Balancing

1.  **Create a Machine Image** of the first frontend instance.
2.  **Launch a new instance** with this image.
3.  **Connect** to the second frontend instance.
4.  **Configure Nginx on the second instance**:
      * Navigate to the frontend directory: `cd TravelMemory/frontend`.
      * Open the Nginx configuration: `sudo nano /etc/nginx/sites-available/default`.
      * Update the `proxy_pass` to include the **public IP of the second instance**.
5.  **Test and Reload Nginx**:
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```
6.  **Run the Frontend Application**:
      * Run: `sudo npm start`.
7.  **Verify Application**[:
      * Navigate to `http://<-second-frontend-Public-IP>` in a web browser to verify the application is running[.

-----

## Load-Balancer Configuration

1.  **Create Frontend Target Group** and attach the two frontend servers.
2.  **Create Backend Target Group** and attach the two backend servers.
3.  **Create Application Load Balancers (ALB)**:
      * Create one ALB for the backend and one for the frontend.
      * Attach the respective target groups.
4.  **Update Security Groups** for the servers:
      * Add the **Security Group ID of the Backend ALB** to the inbound rule of the backend servers for **TCP at port 3000 and HTTP at port 80**.
      * Add the **Security Group ID of the Frontend ALB** to the inbound rule of the frontend servers to allow **TCP at port 3000**.

-----

## Nginx Configuration for Reverse Proxy with ALB

#### Backend Servers (Identical changes for both)

1.  Update the Nginx configuration (`/etc/nginx/sites-available/default`):
      * Set the `server_name` field to your custom domain, e.g., `server_name api.shrijatin.site`.

#### Frontend Servers (Identical changes for both)

1.  Update the Nginx configuration (`/etc/nginx/sites-available/default`):
      * Set the `server_name` to your custom CNAME field, e.g., `server_name shrjatin.site www.shrijatin.site`.
2.  **Update `TravelMemory/frontend/src/url.js`**:
      * Set the backend URL to `"http://api.<customdomain.com>"`, e.g., `"http://api.shrijatin.site"`.
3.  **Reload Nginx** for all instances and **start the servers**.

-----

## Cloudflare Configuration

Configure the following **CNAME DNS Records** in Cloudflare:

| Type | Name | Content |
| :--- | :--- | :--- |
| CNAME | `api` | `<backend-ALB-DNS>` |
| CNAME | `shrijatin.site` | `<frontend-ALB-DNS>` |
| CNAME | `www` | `shrijatin.site` |

The application is now successfully deployed on the custom domain.
