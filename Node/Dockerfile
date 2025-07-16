# ------------ Stage 1: Builder ------------
    FROM node:22-alpine AS builder

    # Set the working directory
    WORKDIR /app
    
    # Install dependencies
    COPY package*.json ./
    RUN npm ci
    
    # Copy the rest of the application code
    COPY . .
    
    
    # ------------ Stage 2: Runner ------------
    FROM node:22-alpine
    
    # Set the working directory
    WORKDIR /app
    
    # Copy only the built app and node_modules from builder
    COPY --from=builder /app /app
    
    # Expose the port the app runs on
    EXPOSE 5003
    
    
    # Start the server
    CMD ["npm", "start"]