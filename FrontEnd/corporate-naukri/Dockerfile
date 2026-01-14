# Multi-stage Dockerfile for building the Angular app and serving via nginx
# Works when build context is:
# 1. The frontend folder (FrontEnd/corporate-naukri): COPY . . copies the frontend
# 2. The repo root: uses a RUN command to find and verify files exist in subdirs

FROM node:20-bullseye-slim AS build
WORKDIR /app

# Install minimal build tools required by some native dependencies
RUN apt-get update \
	&& apt-get install -y --no-install-recommends git python3 build-essential ca-certificates \
	&& rm -rf /var/lib/apt/lists/*

# Copy all from context (handles both frontend-folder and repo-root contexts)
COPY . .

# If we're at repo root, files are in FrontEnd/corporate-naukri/; move them to /app
RUN if [ -d FrontEnd/corporate-naukri ] && [ ! -f package.json ]; then \
  echo "=== Detected repo-root context, moving files ===" && \
  cp -r FrontEnd/corporate-naukri/* . && \
  ls -la; \
fi

# Check if package.json exists
RUN if [ ! -f package.json ]; then echo "ERROR: package.json not found in /app"; ls -la; exit 1; fi

# Install dependencies (includes devDependencies required for build)
RUN npm ci --legacy-peer-deps

# Debug: verify node_modules and @angular/cli are present
RUN echo "=== Node and npm versions ===" \
  && node -v && npm -v \
  && echo "=== Checking @angular/cli ===" \
  && ls -la node_modules/@angular/cli/bin/ || echo "WARNING: @angular/cli not found in node_modules" \
  && echo "=== Angular CLI version ===" \
  && npx ng version || echo "ng command not available"

# Verify nginx.conf exists
RUN if [ ! -f nginx.conf ]; then echo "ERROR: nginx.conf not found"; ls -la; exit 1; fi

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

# Provide the nginx config for SPA routing (copied from build stage)
COPY --from=build /app/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
