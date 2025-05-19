# Stage 1: Build the frontend using Node
FROM node:lts-alpine3.20 AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Stage 2: Serve the app using NGINX
FROM nginx:alpine

# Remove default nginx config if exists
RUN rm /etc/nginx/conf.d/default.conf

# Copy built app to nginx html directory
COPY --from=build /app/dist/ /usr/share/nginx/html

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

# Start NGINX
CMD ["nginx", "-g", "daemon off;"]
