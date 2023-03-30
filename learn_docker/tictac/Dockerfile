FROM node:18-alpine
WORKDIR /app
# Coppy source code outside in /app: first dot: current host (tic tac), second dot current container (app)
COPY . .
RUN npm install
EXPOSE 5173
# Run command inside container
CMD ["npm", "run", "dev"]
# Another: ENTRYPOINT ENTRYPOINT [ "start.sh" ]