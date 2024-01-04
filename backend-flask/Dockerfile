# Use the official Python 3.10 image with a minimal Debian (buster) base
FROM python:3.10-slim-buster

# Set the working directory inside the container
WORKDIR /backend-flask

# Copy the requirements.txt file from the local directory to the container
COPY requirements.txt requirements.txt

# Install Python dependencies listed in requirements.txt
RUN pip3 install -r requirements.txt

# Copy all files from the local directory to the container
COPY . .

# Set the environment variable for Flask to run in development mode
ENV FLASK_ENV=development

# Expose the port specified by the environment variable "PORT"
EXPOSE ${PORT}

# Specify the default command to run when the container starts
CMD [ "python3", "-m", "flask", "run", "--host=0.0.0.0", "--port=4567" ]
