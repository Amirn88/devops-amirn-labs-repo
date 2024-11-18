# Dockerfile for Flask
FROM python:3.9-alpine

WORKDIR /app

# Install system dependencies (optional, adjust based on your requirements)
RUN apk update && apk add --no-cache gcc musl-dev libffi-dev

# Copy requirements.txt and install Python dependencies
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN chmod +x app.py


EXPOSE 8000

CMD ["python3", "app.py"]

