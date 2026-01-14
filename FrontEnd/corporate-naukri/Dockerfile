# Multi-stage Dockerfile for building the Angular app and serving via nginx

FROM node:18-alpine AS build
WORKDIR /app

# Install build dependencies
COPY package.json package-lock.json* ./
RUN npm install --legacy-peer-deps

# Copy source and build
COPY . .
RUN npm run build -- --configuration=production

FROM nginx:stable-alpine

# Copy built site from the previous stage
COPY --from=build /app/dist/corporate-naukri /usr/share/nginx/html

# Optional: provide an nginx config for SPA routing
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
