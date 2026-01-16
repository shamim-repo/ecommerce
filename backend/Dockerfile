FROM node:20-alpine AS base

WORKDIR /app

COPY package.json yarn.lock ./

# Install dependencies
RUN yarn install

COPY . .

# Build the admin UI (if applicable for your setup, Medusa 1.x admin is often built separately or plugin embedded)
# For Medusa 2.0 or if you have admin plugin:
RUN yarn build

FROM node:20-alpine AS production

WORKDIR /app

COPY --from=base /app/node_modules ./node_modules
COPY --from=base /app/dist ./dist
COPY --from=base /app/package.json ./package.json

# Copy other necessary files like medusa-config.js
COPY --from=base /app/medusa-config.js ./medusa-config.js
# COPY --from=base /app/.env ./.env # Secrets should be injected by Dokploy, not copied

# Expose port
EXPOSE 9000

# Start command
# We use a shell script or direct command. 
# Migrations should ideally be run in a separate step or init container, but for simplicity:
CMD ["yarn", "start"]
