# Use official Python image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy files
COPY requirement.txt .
RUN pip install --no-cache-dir -r requirement.txt

COPY . .

# Expose port
EXPOSE 5000

# Start app
CMD ["python", "app.py"]
