
![1-Cheat-Sheet-Images-Containers](https://user-images.githubusercontent.com/12442613/178245392-77e48380-bc52-4eef-983b-39bbcbc70390.jpg)

### in layer 4 `RUN` will run in image build, `CMD` will run after creation of container

Dockerfile
```dockerfile
FROM node:12

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 80

CMD ["node", "server.js"]

```

` docker build . ` to build image

run from the docker image with expose a port <br>
` docker run -p 8000:80 IMAGE_NAME `


start/ stop Containers <br>
` docker start CONTAINER_NAME`

` docker stop CONTAINER_NAME`


------------------------------
`docker run -p 8000:80 IMAGE_NAME ` starts in attached mode `-d` to deattached mode
<br>
`docker start CONTAINER_NAME` starts in deattached mode `-a` to attached mode


interactive mode for taking input in terminal

`docker run -it -p 8000:80 IMAGE_NAME `  <br>

` docker start -a -i CONTAINER_NAME`


to see logs in deattached mode 
`docker logs CONTAINER_NAME`

to see continuos log in deattached mode
`docker logs -f CONTAINER_NAME`

------------------------

### copy to/from container
` docker cp dummy/. CONTAINER_NAME:/test` <br>

` docker cp CONTAINER_NAME:/test dummy `


![image](https://user-images.githubusercontent.com/12442613/194402154-dd71d72a-0a86-4b62-9be6-a0116a1a0c31.png)

![image](https://user-images.githubusercontent.com/12442613/194402296-2b8064b3-549d-46ce-b195-a16e5f04cb46.png)


