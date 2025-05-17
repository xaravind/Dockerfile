# Stage 1: Build the Node.js application
FROM node:lts-alpine3.20 AS build

WORKDIR /app

# Copy only package files first for efficient caching
COPY package*.json ./

RUN npm install

# Copy the rest of the source code
COPY . .

RUN npm run build

# Stage 2: Serve the built app using Apache HTTP Server
FROM httpd:alpine3.21

# Copy the built app to the Apache web root
COPY --from=build /app/dist/ /usr/local/apache2/htdocs/