FROM python:3.8 

# Set working directory 
WORKDIR /app 

# Copy project files
 COPY . /app 

# Install dependencies 
RUN pip install --no-cache-dir -r requirements.txt 

# Set Flask environment variables 
ENV FLASK_APP=app.py \ 
        FLASK_RUN_HOST=0.0.0.0 

# Expose port 
EXPOSE 5000 

# Run Flask 
CMD ["flask", "run"] 
