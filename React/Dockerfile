# Build stage
FROM node:22-alpine AS build

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Build the app
RUN npm run build

# Production stage
FROM node:22-alpine

# Set working directory
WORKDIR /app

# Install serve
RUN npm install -g serve

# Copy build files from build stage
COPY --from=build /app/dist ./build

# Expose port 5173
EXPOSE 5002

# Start the application
CMD ["serve", "-s", "build", "-l", "5002"]