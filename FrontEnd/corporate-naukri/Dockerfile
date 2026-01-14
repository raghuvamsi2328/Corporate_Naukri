# Multi-stage Dockerfile for building the Angular app and serving via nginx

FROM node:18-bullseye-slim AS build
WORKDIR /app

# Install minimal build tools required by some native dependencies
RUN apt-get update \
	&& apt-get install -y --no-install-recommends git python3 build-essential ca-certificates \
	&& rm -rf /var/lib/apt/lists/*

# Copy package files and install dependencies deterministically
COPY package*.json ./
RUN npm ci --legacy-peer-deps

# Copy project files and build production bundle
COPY . .
ENV CI=true
# Increase Node's heap size during the Angular build to avoid OOM failures inside the container
ENV NODE_OPTIONS=--max_old_space_size=4096
RUN npm run build -- --configuration=production

FROM nginx:stable

# Copy built site from the previous stage
COPY --from=build /app/dist/corporate-naukri /usr/share/nginx/html

# Provide the nginx config for SPA routing
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
