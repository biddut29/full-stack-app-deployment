
# Full-Stack Application Setup Documentation

This documentation provides a step-by-step guide to setting up a full-stack application with a Node.js backend and a frontend served by Nginx.

---

## 1. Project Structure

The project is structured as follows:

```
my-fullstack-app/
├── backend/
│   ├── node_modules/
│   ├── package.json
│   ├── package-lock.json
│   └── server.js
└── frontend/
    ├── node_modules/
    ├── public/
    ├── src/
    ├── package.json
    └── vite.config.js
```

---

## 2. Backend Setup

### 2.1. Install Node.js and npm

1. **Install `nvm` (Node Version Manager):**
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
   ```

2. **Load `nvm` without restarting the shell:**
   ```bash
   \. "$HOME/.nvm/nvm.sh"
   ```

3. **Install Node.js v22:**
   ```bash
   nvm install 22
   ```

4. **Verify the installation:**
   ```bash
   node -v  # Should print "v22.14.0"
   npm -v   # Should print "10.9.2"
   ```

---

### 2.2. Set Up the Backend

1. **Navigate to the `backend` directory:**
   ```bash
   cd my-fullstack-app/backend
   ```

2. **Install dependencies:**
   ```bash
   npm install express cors
   ```

3. **Create `server.js`:**
   ```javascript
   const express = require("express");
   const cors = require("cors");

   const app = express();
   app.use(cors());
   app.use(express.json());

   app.get("/", (req, res) => {
       res.send({ message: "Hello from Node.js Backend!" });
   });

   const PORT = 5000;
   app.listen(PORT, () => {
       console.log(`Server running on http://localhost:${PORT}`);
   });
   ```

4. **Run the server:**
   ```bash
   node server.js
   ```

5. **Use PM2 for process management:**
   ```bash
   sudo npm install -g pm2
   pm2 start server.js --name "backend"
   pm2 save
   pm2 startup
   ```

---

## 3. Frontend Setup

### 3.1. Navigate to the `frontend` Directory

1. **Go to the `frontend` directory:**
   ```bash
   cd ../frontend
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Update `package.json`:**
   Change the `dev` script to:
   ```json
   "dev": "vite --host 0.0.0.0"
   ```

4. **Run the development server:**
   ```bash
   npm run dev
   ```

5. **Build the frontend for production:**
   ```bash
   npm run build
   ```

---

## 4. Nginx Configuration

### 4.1. Install Nginx

1. **Install Nginx:**
   ```bash
   sudo apt install nginx -y
   ```

2. **Create an Nginx configuration file:**
   ```bash
   sudo nano /etc/nginx/sites-available/frontend
   ```

   Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name your_domain_or_ip;

       root /home/azureuser/my-fullstack-app/frontend/dist;
       index index.html;

       location / {
           try_files $uri $uri/ /index.html;
       }

       location /api {
           proxy_pass http://localhost:5000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

3. **Enable the configuration:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
   ```

4. **Set permissions for the `dist` folder:**
   ```bash
   sudo chcon -Rt httpd_sys_content_t /home/azureuser/my-fullstack-app/frontend/dist
   ```

5. **Test and reload Nginx:**
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   sudo systemctl enable nginx
   ```

---

## 5. Access the Application

- **Frontend:** Open your browser and navigate to `http://your_domain_or_ip`.
- **Backend:** The backend API is accessible at `http://your_domain_or_ip/api`.

---

## 6. Troubleshooting

- **Nginx fails to start:**
  - Check the Nginx error logs: `sudo tail -f /var/log/nginx/error.log`.
  - Ensure the `dist` folder exists and has the correct permissions.

- **PM2 process not running:**
  - Check PM2 logs: `pm2 logs backend`.
  - Restart the process: `pm2 restart backend`.

---
