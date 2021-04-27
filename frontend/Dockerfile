# to be served by NGINX
FROM node:alpine as builder
USER node
RUN mkdir -p /home/node/app
WORKDIR /home/node/app
COPY --chown=node:node ./package.json ./
RUN npm install
COPY --chown=node:node ./ ./
# EXPOSE 3000
# CMD [ "npm", "start" ]
RUN npm run build
FROM nginx
COPY --from=builder /home/node/app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx","-g","daemon off;"]