# Multi-stage Dockerfile for building the Angular app and serving via nginx

ARG APP_DIR=.
FROM node:18-bullseye-slim AS build
WORKDIR /app

# Install minimal build tools required by some native dependencies
RUN apt-get update \
	&& apt-get install -y --no-install-recommends git python3 build-essential ca-certificates \
	&& rm -rf /var/lib/apt/lists/*

# Copy package files and install dependencies deterministically
# APP_DIR is a build-arg so this Dockerfile works whether the build context is
# the frontend folder (default) or the repository root (set APP_DIR=FrontEnd/corporate-naukri)
COPY ${APP_DIR}/package*.json ./
# Ensure devDependencies (like @angular/cli) are installed for the build
ENV NODE_ENV=development
RUN npm ci --legacy-peer-deps

# Copy project files and build production bundle
COPY ${APP_DIR}/ ./
ENV CI=true
# Increase Node's heap size during the Angular build to avoid OOM failures inside the container
ENV NODE_OPTIONS=--max_old_space_size=4096
# Use npx to ensure the Angular CLI is available even if global ng is not installed.
# This will prefer the local @angular/cli from node_modules, or fetch it if missing.
RUN npx -y -p @angular/cli ng build --configuration=production
# Set NODE_ENV back to production for the resulting image
ENV NODE_ENV=production

FROM nginx:stable

# Copy built site from the previous stage
COPY --from=build /app/dist/corporate-naukri /usr/share/nginx/html

# Provide the nginx config for SPA routing
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
