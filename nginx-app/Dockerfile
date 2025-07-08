FROM nginx:alpine

# Remove default nginx files
RUN rm -rf /usr/share/nginx/html/*

# Copy everything from current directory (where Dockerfile is) into nginx's html folder
COPY . /usr/share/nginx/html/

EXPOSE 80
