[9:43 pm, 26/01/2025] Ahmed Sameer Best Friend : Create a custom image with the following specs:

● Base image: postman/newman:5.3-alpine
● Update/upgrade the image packages to the latest.
● Globally Install node modules: newman-reporter-csvallinone
● Install extra packages: curl zip ping
● Set the DNS explicitly to 8.8.8.8 & 1.1.1.1
● Remove the installation cache.
● Set an environment variable NODE_PATH to have the value /usr/local/lib/node_modules
● Set working directory to /etc/newman
● Set the entry point to newman
[9:43 pm, 26/01/2025] Ahmed Sameer Best Friend : your next task. create this docker image


# Use a specific version of the base image
FROM postman/newman:5.3-alpine

# Set environment variables upfront
ENV NODE_PATH=/usr/local/lib/node_modules

# Update and install required packages in a single layer
RUN apk update && apk upgrade --no-cache && \
    apk add --no-cache curl zip && \
    npm install -g newman-reporter-csvallinone && \
    rm -rf /root/.npm

# Specify an alternative DNS resolver if needed during container runtime
CMD ["sh", "-c", "echo 'nameserver 8.8.8.8' > /etc/resolv.conf && echo 'nameserver 1.1.1.1' >> /etc/resolv.conf && newman"]
