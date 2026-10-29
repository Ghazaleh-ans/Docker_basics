# build step
FROM node:20 AS node-builder
WORKDIR /app
COPY . .
RUN npm ci
RUN npm run build

#production step
FROM nginx:alpine
COPY --from=node-builder /app/dist /usr/share/nginx/html
