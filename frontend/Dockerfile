FROM alpine:latest
RUN apk update && \
    apk add curl && \
    apk add bash
RUN apk add nodejs \
    && apk add npm
RUN node -v
RUN npm -v
WORKDIR /app
RUN npm install -g @vue/cli
RUN vue create --default tic-tac-toe
RUN rm -r /app/tic-tac-toe/src/App.vue
COPY /src/App.vue /app/tic-tac-toe/src/
WORKDIR /app/tic-tac-toe
RUN npm install axios
EXPOSE 8080
CMD ["npm", "run", "serve"]
