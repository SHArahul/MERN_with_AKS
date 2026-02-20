FROM node:18-alpine

WORKDIR /app

# Copy package.json and install dependencies first (caching)
COPY package*.json ./
RUN npm install --production

# Copy all code
COPY . .

# Expose service port from .env (by default these services use env PORT)
EXPOSE 3001

# Run app
CMD ["npm", "start"]