FROM nginx:latest

WORKDIR /usr/share/nginx/html

COPY jenkins/lab3/static_website/ .

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
