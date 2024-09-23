---
title: Vite tricks
tags: [Frontend]

---

Vite tricks for runtime ENV vars (from chatGPT)
===



You are correct that once a React app is statically built, the environment variables are baked into the build and cannot be changed without rebuilding the app. This is because during the build process, environment variables are inlined into the JavaScript bundle.

However, for Docker-based deployments, you can use a workaround to inject environment variables at runtime. Here's a general approach:

1. Instead of baking the environment variables into the build, modify your application to read the environment variables from the runtime environment (e.g., window.env in the browser).
2. In your Docker setup, use a small server-side script that runs when the container starts, which generates a JavaScript file containing the environment variables based on the current environment variables of the container.
3. Serve this generated JavaScript file before your application loads. Here's an example of how you might implement this:

First, create a placeholder for the environment variables in your public folder, e.g., public/env-config.js:

```js=
window.env = {
  REACT_APP_API_URL: '',
  // ... other variables
};
```

In your React application, access the environment variables using window.env.REACT_APP_API_URL instead of process.env.REACT_APP_API_URL.

Then, in your Dockerfile, add a step that will replace the env-config.js with actual values when the container starts:

```dockerfile!
# Build stage
FROM node:alpine as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY env.sh /
COPY .env /
RUN chmod +x /env.sh

# When the container starts, env.sh will run and inject environment variables
CMD ["/bin/sh", "-c", "/env.sh && nginx -g 'daemon off;'"]
```


Create an env.sh script that will generate the env-config.js file with the actual environment variable values when the container starts:

```bash
#!/bin/sh
# Generate env-config.js with current environment variables
echo "window.env = {" > /usr/share/nginx/html/env-config.js
printenv | grep -v "no_proxy" | while read -r line; do
  echo "  ${line}," >> /usr/share/nginx/html/env-config.js
done
echo "};" >> /usr/share/nginx/html/env-config.js
```

Make sure you chmod +x env.sh to make it executable.

Now, when you run your Docker container, you can pass environment variables like this:

```bash
docker run -e REACT_APP_API_URL=https://api.example.com -p 8080:80 your-image-name
```

The env.sh script runs when the container starts, reading the current environment variables, and generates the env-config.js that the React app can use. The application will then read the runtime configuration from window.env.