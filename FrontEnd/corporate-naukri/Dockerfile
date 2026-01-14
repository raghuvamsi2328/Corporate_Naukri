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

# Install dependencies (includes devDependencies required for build)
RUN npm ci --legacy-peer-deps

# Debug: verify node_modules and @angular/cli are present
RUN echo "=== Node and npm versions ===" \
  && node -v && npm -v \
  && echo "=== Checking @angular/cli ===" \
  && ls -la node_modules/@angular/cli/bin/ || echo "WARNING: @angular/cli not found in node_modules" \
  && echo "=== Angular CLI version ===" \
  && npx ng version || echo "ng command not available"

# Copy project files and build production bundle
COPY ${APP_DIR}/ ./

# Build Angular app using npm script (defined in package.json)
ENV CI=true
ENV NODE_OPTIONS=--max_old_space_size=4096
RUN echo "=== Starting Angular build ===" \
  && npm run build -- --configuration=production

RUN echo "=== Build completed, checking dist ===" \
  && ls -la dist/

FROM nginx:stable

# Copy built site from the previous stage
COPY --from=build /app/dist/corporate-naukri /usr/share/nginx/html

# Provide the nginx config for SPA routing
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
