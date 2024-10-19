### Project Introduction: Full Stack Web Application with Docker and Microservices

This project is a **full-stack web application** built using a microservices architecture. It comprises three core components:

1. **Frontend (Web)**: A user-facing application, built with modern JavaScript frameworks like React, providing a dynamic and responsive user interface. It serves the web pages and interacts with the backend API to fetch or submit data.
   
2. **Backend (API)**: A RESTful API built with Node.js (or any backend framework) that handles business logic, processes requests, and interacts with the database. This API manages authentication, data processing, and routes requests between the frontend and the database.

3. **Database (MongoDB)**: A NoSQL database that stores and manages data. MongoDB is used to persist application data, such as user information, application state, and other relevant entities.

### Key Features
- **Dockerized Services**: Each component (frontend, backend, and database) is containerized using Docker. This ensures consistency across different environments, simplifying deployment and scaling.
  
- **Microservices Architecture**: The application is designed with a microservices approach, meaning the frontend, backend, and database are independent services that communicate with each other via network requests.

- **Continuous Integration**: Docker and Jenkins are integrated for automated builds, testing, and deployment. Every code change is automatically tested and deployed in isolated environments.

- **Port Mapping**: The frontend is accessible via `localhost:3000`, while the backend API runs on `localhost:3001`. Both services can communicate with the MongoDB database through an internal network.

### Technologies Used
- **Frontend**: React (or similar) for building a responsive and dynamic UI.
- **Backend**: Node.js/Express.js to create a RESTful API.
- **Database**: MongoDB for storing and retrieving application data.
- **Docker**: For containerization, ensuring consistent environments.
- **Jenkins**: For continuous integration and deployment.
- **Docker Compose**: To orchestrate multi-container Docker applications.

### How It Works
1. **Frontend**: The user interacts with the frontend via a web browser (served on port 3000).
2. **Backend API**: The frontend communicates with the backend API (running on port 3001) to handle user requests and provide data.
3. **MongoDB**: The backend retrieves and stores data in MongoDB, with all services running inside Docker containers for seamless interaction.

---

This introduction sets the stage for your project by highlighting the key components, technologies, and how they fit together in a Dockerized, microservices setup. Let me know if you'd like to refine or expand this!

## Backend Docker file 
```bash
FROM node:10-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3001:3001
CMD ["node", "server.js"]
```
Here’s a simple line-by-line explanation of why we're using each line in this Dockerfile:
### 1. `FROM node:10-alpine`
-  We need Node.js to run our app. This uses a small, efficient version of Linux (Alpine) with Node.js already installed, saving space and setup time.

### 2. `WORKDIR /usr/src/app`
-  Sets a folder inside the container where all our app files will go. It keeps everything organized in the container.

### 3. `COPY package*.json ./`
-  Copies `package.json` and `package-lock.json` into the container. These files list our app's dependencies, so Docker needs them to install the right packages.

### 4. `RUN npm install`
-  Installs all the dependencies (packages) our app needs to run, based on the `package.json` file. 

### 5. `COPY . .`
-  Copies all our app's code into the container. This lets the container have everything it needs to run our app.

### 6. `EXPOSE 3001:3001`
-  Opens port 3001 inside the container and makes it available outside (on our computer). This lets us access the app from a browser (localhost:3001).

### 7. `CMD ["node", "server.js"]`
-  Tells Docker to run our app by starting the `server.js` file with Node.js. This is the main file that starts our application.

### Simple Overview:
Each line prepares the container to:
- Use Node.js.
- Set up a working folder.
- Install necessary packages.
- Copy our app files.
- Open a port to access the app.
- Start our app.

## Frontend Docker file 
```bash
FROM node:14-alpine
WORKDIR /app
# add '/app/node_modules/.bin' to $PATH
ENV PATH /app/node_modules/.bin:$PATH
# install application dependencies
COPY package*.json ./
RUN npm install
# RUN npm install react-scripts -g
# copy app files
COPY . .
EXPOSE 3000:3000
CMD ["npm", "start"]
```
Here’s a simple line-by-line explanation of why we're using each line in this Dockerfile:
### 1. `FROM node:14-alpine`
-  We're using Node.js version 14 with the lightweight Alpine Linux. This ensures we have the right version of Node.js in a small and efficient environment for our app.

### 2. `WORKDIR /app`
-  This sets the working directory inside the container to `/app`. All future commands will run in this folder. It's a way to keep our files organized.

### 3. `ENV PATH /app/node_modules/.bin:$PATH`
-  This adds `/app/node_modules/.bin` to the system’s `$PATH` environment variable. This means any binaries installed by npm (like project-related scripts) will be accessible from anywhere in the container without needing to specify the full path.

### 4. `COPY package*.json ./`
-  This copies `package.json` and `package-lock.json` from our local system into the container’s working directory. These files contain information about the app’s dependencies.

### 5. `RUN npm install`
-  This installs the application dependencies listed in the `package.json` file. It's a key step to ensure all required Node.js packages are set up before the app runs.

### 6. `COPY . .`
-  This copies all the app’s source code from our local machine to the container’s working directory (`/app`). This ensures the container has access to the full application code.

### 7. `EXPOSE 3000:3000`
-  This opens port 3000 inside the container and maps it to port 3000 on the host machine. This lets us access the app from our local machine (through a browser) at `localhost:3000`.

### 8. `CMD ["npm", "start"]`
-  This tells Docker to run the command `npm start` when the container starts. This is typically the command to start the development server for a Node.js or React app.

### Simple Overview:
Each line in this Dockerfile prepares the container to:
- Use Node.js (version 14).
- Set up a working directory (`/app`).
- Install necessary packages.
- Add the npm binaries to the system's path.
- Copy the application code.
- Open a port (3000) to access the app.
- Start the app with `npm start`.

## For MongoDB, we don't need a Dockerfile because we can directly pull the official MongoDB image. The necessary attributes for its configuration (such as network settings, ports, etc.) are specified in the Docker Compose file.

## Docker Compose file
```bash
version: "3.8"
services:
  web:
    build: ./frontend
    depends_on:
      - api
    ports:
      - "3000:3000"
    networks:
      - network-backend
  api:
    build: ./backend
    depends_on:
      - mongo
    ports:
      - "3001:3001"
    networks: 
     - network-backend
  
  mongo:
    image: mongo
    restart: always
    volumes: 
      - mongodb_data:/data/db
    environment: 
      MONGODB_INITDB_ROOT_USERNAME: username
      MONGODB_INITDB_ROOT_PASSWORD: password
    networks: 
     - network-backend

networks:
  network-backend:

volumes: 
  mongodb_data:
```
Here’s a simple explanation of the Docker Compose file:
### 1. `version: "3.8"`
-  Specifies the version of Docker Compose being used. Version 3.8 is a stable version that supports all the needed features.

### 2. `services:` 
-  This section defines the different services (containers) we want to run. In this case, it's `web`, `api`, and `mongo`.

### 3. `web:` 
-  This is the frontend service. It builds the frontend app from the `./frontend` folder.
- **`depends_on:`**: This tells Docker that the `web` service depends on the `api` service, so `api` must start first.
- **`ports:`**: Maps port 3000 in the container to port 3000 on the host machine, allowing access to the frontend at `localhost:3000`.
- **`networks:`**: Connects the `web` service to the `network-backend` network so it can communicate with other services.

### 4. `api:` 
- This is the backend service. It builds the backend app from the `./backend` folder.
- **`depends_on:`**: The `api` service depends on `mongo`, so `mongo` must start first.
- **`ports:`**: Maps port 3001 in the container to port 3001 on the host machine, allowing access to the API at `localhost:3001`.
- **`networks:`**: Connects the `api` service to the `network-backend` network so it can communicate with other services.

### 5. `mongo:` 
-  This is the MongoDB service. It pulls the MongoDB image from Docker Hub to set up a MongoDB database.
- **`restart: always`**: Ensures that the MongoDB service restarts automatically if it crashes or stops.
- **`volumes:`**: The MongoDB data is stored in a persistent volume called `mongodb_data` to keep the database data even if the container is removed.
- **`environment:`**: Sets the MongoDB root username and password for secure access.
- **`networks:`**: Connects the MongoDB service to the `network-backend` network.

### 6. `networks:` 
-  Defines a custom network called `network-backend` that connects all the services (`web`, `api`, `mongo`) so they can communicate with each other internally.

### 7. `volumes:` 
- **Why**: Defines a persistent volume `mongodb_data` to store MongoDB's data, so the data is not lost when the container is removed.

### Simple Overview:
- **`web`**: Frontend service (React or similar), runs on `localhost:3000`.
- **`api`**: Backend service (Node.js or similar), runs on `localhost:3001`.
- **`mongo`**: MongoDB database, stores its data in a volume.
- All services are connected on a custom network (`network-backend`) so they can communicate.
