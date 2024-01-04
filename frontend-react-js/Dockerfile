# Use the official Node.js 16.18 image as the base image
FROM node:16.18

# Set the environment variable PORT to 3000
ENV PORT=3000

# Copy the contents of the local directory to /frontend-react-js in the container
COPY . /frontend-react-js

# Set the working directory to /frontend-react-js
WORKDIR /frontend-react-js

# Install Node.js dependencies using npm
RUN npm install

# Expose the specified port (3000) for the application
EXPOSE ${PORT}

# Define the default command to run when the container starts
CMD [ "npm", "start" ]

