# My Experience Implementing the MEAN Stack on AWS EC2 with a Self-Hosted MongoDB

In this project i am going to share my experience why setting up a MEAN Stack application on an AWS EC2 Instance for production, using a self-hosted MongoDB database. 

The **MEAN stack** is a popular web development technology stack used for building dynamic and robust web applications. MEAN is an acronym that stands for the four main technologies that comprise the stack:

1. **MongoDB**: A NoSQL database that stores data in a flexible, JSON-like format. It allows for easy data retrieval and storage and is designed to handle large volumes of unstructured data.

2. **Express.js**: A web application framework for Node.js that simplifies the process of building web applications and APIs. It provides a robust set of features for web and mobile applications, such as routing, middleware support, and handling HTTP requests and responses.

3. **Angular**: A front-end web application framework developed by Google. It allows developers to build single-page applications (SPAs) using HTML, CSS, and TypeScript (a superset of JavaScript). Angular provides a rich set of tools for building dynamic user interfaces and managing the application's state.

4. **Node.js**: A JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js enables developers to run JavaScript on the server side, allowing for building scalable and efficient web applications. It uses an event-driven, non-blocking I/O model, which makes it lightweight and suitable for data-intensive real-time applications.

### **How the MEAN Stack Works**
- **Client-Side (Angular)**: The user interacts with the front-end application, which is built using Angular. The application sends requests to the server.
- **Server-Side (Node.js and Express.js)**: The server receives requests from the Angular front end and processes them using Express.js. It may interact with the MongoDB database to retrieve or store data.
- **Database (MongoDB)**: MongoDB stores the application's data in a flexible JSON-like format, allowing for easy manipulation and retrieval of information.

### **Key Features of the MEAN Stack**
1. **Full-Stack JavaScript**: The MEAN stack uses JavaScript for both client-side and server-side development, allowing for a more streamlined development process and easier code sharing between the front end and back end.

2. **Single-Page Applications (SPAs)**: Angular enables the development of SPAs, providing a smooth user experience by dynamically updating the content without reloading the entire page.

3. **Scalability**: The stack is designed to handle large-scale applications, making it suitable for applications that require a high level of performance and scalability.

4. **Rapid Development**: The combination of technologies in the MEAN stack allows for rapid prototyping and development of applications.

5. **JSON Everywhere**: The data is stored in JSON format in MongoDB and exchanged between the client and server as JSON, making data manipulation easier.

### **Advantages of the MEAN Stack**
- **Uniform Language**: Using JavaScript across the entire stack simplifies the development process, as developers can work on both the front end and back end without switching languages.
- **Community Support**: Each component of the MEAN stack has a large and active community, providing extensive resources, libraries, and support.
- **Flexibility**: The stack is highly flexible and allows for easy integration with other technologies and frameworks.
- **Performance**: Node.js and MongoDB provide high performance, especially for I/O-heavy applications.

### **Disadvantages of the MEAN Stack**
- **Steep Learning Curve**: Developers unfamiliar with any of the technologies may face a learning curve when getting started with the stack.
- **Complexity**: Managing the entire stack can be complex, especially for large applications with multiple components.
- **No SQL**: MongoDB's NoSQL approach may not be suitable for all types of applications, especially those that require complex transactions or relationships.

### **Conclusion**
The MEAN stack is a powerful and efficient technology stack for developing modern web applications. Its use of JavaScript across the full stack simplifies development and enhances productivity, making it a popular choice among developers for building dynamic and scalable web applications.

Before enbacking on this project it is neccesary to have your AWS EC2 instance running and a MEAN stack application ready to be deployed.   The EC2 instance should have ports 5000 and 27017 open 

# Installing MongoDB

to install MongoDB, i followed this proceedure 

+ Imported the public key for the MongoDB package:
```
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
```
+ I created a MongoDB source list file:
 ```
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
  ```
+ I Updated the package list and installed MongoDB:

```
sudo apt-get update
sudo apt-get install -y mongodb-org
```
+ I Started MongoDB and enabled it to start on boot:
```
sudo systemctl start mongod
sudo systemctl enable mongod
```
+ I verified the installation:
`sudo systemctl status mongod`

# Configuring MongoDB

It was neccasary that i configured MongoDB for production 

+ i configured the remote access  using this command
+ 
` sudo nano /etc/mongod.conf`

i Changed the bindIp setting:

```
net:
  bindIp: 0.0.0.0
```
![image](https://github.com/user-attachments/assets/e0114b34-3978-49c9-9fd3-c784ace48a19)

The ensence of this was to allow remote connections 

+ I secured the Mongodb: Start MongoDB without authentication enabled:

` mongosh `

![image](https://github.com/user-attachments/assets/593652e1-225e-4210-898c-94b1faa9f207)

I Connected to MongoDB and create the admin user: 
```
use admin
```
```
use admin

db.createUser(
    {
        user: "root",
        pwd: "Password.1",
        roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
    }
)

exit

```

![image](https://github.com/user-attachments/assets/fe6b0fba-971b-48f5-9f3e-5d25b3c52068)

Now, edit the MongoDB configuration file (usually /etc/mongod.conf on Linux systems) to enable authentication:
```
sudo nano /etc/mongod.conf
```
Changed the security setting:
```
security:
    authorization: enabled
```
*It is important to enable the authentication for security, without it, anyone could* *potentially access my database.*

Restart the MongoDB service to apply the changes.

```
sudo systemctl restart mongod
```

Setting up a MongoDB user:
```
 mongosh --authenticationDatabase "admin" -u "root" -p
```
```
use sample_book_register_app
db.createUser({
  user: "bookRegisterUser",
  pwd: "Password.1",
  roles: [{ role: "readWrite", db: "sample_book_register_app" }]
})
```
*it is immport that i create a specific user for the application and i gave it a as "PassWord.1"*

The resulting connection string:

![image](https://github.com/user-attachments/assets/a5f7c8c0-94ba-4afa-806c-b26eff307dfd)

# Installing Node.js

i followed this command while installing Node.js
```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt-get install -y nodejs
```
![image](https://github.com/user-attachments/assets/86ba5e7c-4720-4752-b391-2120c7b4a5c2)

To Verified the installation i used 
```
node -v
npm -v
```

![image](https://github.com/user-attachments/assets/ffb0f674-23cc-4a21-8e43-09a5514071d2)

# Setting Up PM2 Process Manager

*Setting up  PM2 was very important for production so i make use of this command*

```
sudo npm install pm2 -g
```
![image](https://github.com/user-attachments/assets/be218d9b-49c5-428c-9edc-33a15c7acdb9)

# Installing and Configuring Nginx 

I set up Nginx as a reverse proxy following this steps 
+ Installed Nginx:
  ```
  sudo apt update
  sudo apt install nginx -y
  ```
![image](https://github.com/user-attachments/assets/0e582909-23e7-40f6-ab0f-7034e6c9d5d1)
+ i Created a custom Nginx configuration:
```
sudo nano /etc/nginx/sites-available/sample_book_register_app
```
Configuration 
```
server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
+ Activated the new configuration and deactivated the default:
```
sudo ln -s /etc/nginx/sites-available/sample_book_register_app /etc/nginx/sites-enabled/
sudo nginx -t
sudo unlink /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```
![image](https://github.com/user-attachments/assets/f6739797-d604-444b-983f-ae42acd8c90c)

# Deploying the MEAN Application
Its finally time to deploy the MEAN application:

+ Install the Angular-cli:
` sudo npm install -g @angular/cli`
+ Clone the MEAN Stack project repository:
`git clone https://github.com/fmanimashaun/sample-book-register-app.git`

+ Set up the frontend:
```
cd sample-book-register-app/client
sudo npm i && sudo npm run build
```

![image](https://github.com/user-attachments/assets/6f82e2ae-d67a-46e9-af56-497bf38b1349)

+ Set up the backend:
```
cd ../backend
sudo npm i
```

![image](https://github.com/user-attachments/assets/358dae5f-bf56-4493-a0af-167b186af0db)

+ Created the .env file inside the backend folder:
`sudo nano .env`

Here is the Content of the folder 
```
PORT=5000
DB=mongodb://bookRegisterUser:Password.1@localhost:27017/sample_book_register_app?authSource=sample_book_register_app
NODE_ENV=production
```
+ Started the application with PM2
  
`pm2 start index.js --name mean-app`

+ Kept the app running and ensured it restarted automatically after crashes or system reboots:
```
pm2 startup
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u <linux user> --hp /home/<linux user>
pm2 save
```
*In my case, ubuntu is the linux user for the EC2 instance.*
![image](https://github.com/user-attachments/assets/14c1d630-f0b0-4c7f-8756-edc0bb6a69f8)
* For the above code to run properly ensure that you are in the backend folder before running the command

# Final Steps and Reflections


+ Checked the process status:
  
```
pm2 logs mean-app
```

+ Accessed the application at http://<aws-ec2-instance-public-ip>.
  
  ![image](https://github.com/user-attachments/assets/5d5bb2aa-4e59-4653-ba62-e677d3dc8f66)


+ Tested the backend API using Postman and Browser:
*sample datas to use:*\
```
[
    {
        "name": "The Pragmatic Programmer",
        "isbn": "978-0201616224",
        "author": "Andrew Hunt, David Thomas",
        "pages": 352
    },
    {
        "name": "Clean Code",
        "isbn": "978-0132350884",
        "author": "Robert C. Martin",
        "pages": 464
    },
    {
        "name": "You Don’t Know JS: Scope & Closures",
        "isbn": "978-1449335588",
        "author": "Kyle Simpson",
        "pages": 98
    }
]
```

+ 

+ 
