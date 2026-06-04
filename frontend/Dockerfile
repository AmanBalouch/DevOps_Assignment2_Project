FROM node:18-alpine AS build

WORKDIR /app

COPY package.json package-lock.json . 

RUN npm install

COPY . .

RUN npm run build

FROM httpd:2.4-alpine

WORKDIR /usr/local/apache2/htdocs

RUN rm -rf ./*

COPY --from=build /app/dist .

EXPOSE 80

CMD ["httpd-foreground"]

EXPOSE 80

CMD ["httpd-foreground"]

