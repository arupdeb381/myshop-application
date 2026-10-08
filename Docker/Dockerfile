
FROM nginx:stable-alpine

LABEL app="devops-demo"
LABEL version="1.0.0"

COPY src/index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
