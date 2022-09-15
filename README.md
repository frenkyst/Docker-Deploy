# Docker-Deploy


1. Install docker terlebih dahulu dengan scrip bash

       nano autoDocker.sh
         
   ![image](https://user-images.githubusercontent.com/40049149/189819171-3692ccee-b0bf-4cfd-9cd0-0857b26e802f.png)
   
       #!/bin/bash

       sudo apt update

       sudo apt install \
         ca-certificates \
         curl \
         gnupg \
         lsb-release

       sudo mkdir -p /etc/apt/keyrings

       curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

       echo \
         "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
         $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

       sudo apt update
       sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

2. Tambahkan permission execute di file yang kita buat tadi search.sh dengan command chmod

       chmod +x autoDocker.sh
       ls -l

   ![image](https://user-images.githubusercontent.com/40049149/189820014-51e4f4f9-00fb-48a2-8f3f-9f8018a6c802.png)

3. Jalankan script dengan command berikut

       ./autoDocker.sh

   ![image](https://user-images.githubusercontent.com/40049149/189820208-d1836bae-f59b-4f29-af60-3a18143bec25.png)

4. Cek versi docker

       docker -v
       docker version
    
   ![image](https://user-images.githubusercontent.com/40049149/189824694-9e9b74ee-2514-42c7-b37f-30dc2c249973.png)

5. Kita akan membuat perintah docker agar tidak menggunakan sudo lagi dengan perintah berikut:

       sudo usermod -aG docker menther

   ![image](https://user-images.githubusercontent.com/40049149/189825815-5672e9ee-2917-4049-871e-8fcf6eab13b4.png)

6. Login ke docker > https://hub.docker.com/

       docker login
       
   ![image](https://user-images.githubusercontent.com/40049149/189826505-55131e58-34d2-4efe-ac6f-2ded4edcb971.png)


## Docker Imange

7. Sekarang kita mendownload mysql

       docker pull mysql

   ![image](https://user-images.githubusercontent.com/40049149/189835008-2ac64f88-f140-4dad-8619-39d55c6e8ab8.png)

8. Cek jikat sudah di download

       docker images
   
   ![image](https://user-images.githubusercontent.com/40049149/189835147-aeb431d4-a195-45bb-aecf-4ab6105a3a09.png)

9. Sekarang clone frontend dan backend nya

       git clone https://github.com/dumbwaysdev/housy-frontend
       git clone https://github.com/dumbwaysdev/housy-backend

   ![image](https://user-images.githubusercontent.com/40049149/189836588-6aa889d4-0326-46c3-9526-20d14d7d947c.png)

10. Install terlebih dahulu docker compose

        sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
        sudo chmod +x /usr/local/bin/docker-compose
        docker compose version

    ![image](https://user-images.githubusercontent.com/40049149/189837166-6e6e23df-de68-4422-af6f-469adcd2a1e1.png)
    ![image](https://user-images.githubusercontent.com/40049149/189837402-d2f8bcdd-e467-47ff-92f8-a78184210c1f.png)

11. Masuk direktory frontEnd 

        cd housy-frontend/
       
    ![image](https://user-images.githubusercontent.com/40049149/189842431-b6a3cf70-63e8-4a80-9a20-3023fac55032.png)

        nano src/config/api.js

    ![image](https://user-images.githubusercontent.com/40049149/189842149-c0472abc-ad02-475d-901b-3d4a55ac187e.png)

12. Download node:10-alpine

        docker pull node:10-alpine

    ![image](https://user-images.githubusercontent.com/40049149/189845376-b275329d-08d1-4bdd-8711-5c82f9514a67.png)

13. Buat file dockerfile di directory frontEnd

        FROM node:10-alpine
        WORKDIR /usr/app
        COPY . .
        RUN npm install
        EXPOSE 3000
        CMD [ "npm", "start" ]

    ![image](https://user-images.githubusercontent.com/40049149/189845897-d3541a0a-00d4-4b99-9af0-280f3f49f42c.png)

14. Buat file docker-compose.yaml untuk frontend dengan isi:

        version: '3.7'
        services:
         frontend:
          build: .
          container_name: fe
          image: menther/housy-fe:stable
          stdin_open: true
          ports:
           - 3000:3000

    ![image](https://user-images.githubusercontent.com/40049149/189848874-ed3b5a40-67c6-4057-ae00-e67789f8aa74.png)

15. Build dan Jalankan compose secara daemon

        docker-compose up -d
        
    ![Screenshot from 2022-09-13 15-19-37](https://user-images.githubusercontent.com/40049149/189850145-bd946097-bcba-4180-bf08-d8da1a09bf6f.png)

16. Ke directory BackEnd edit file config.json

        nano config/config.json

    ![image](https://user-images.githubusercontent.com/40049149/189876567-47a53386-b071-4d77-8a06-c1f8f212925e.png)

    ![Screenshot from 2022-09-13 17-16-18](https://user-images.githubusercontent.com/40049149/189876748-f43a4018-7927-46cc-b8e0-e6397c63a3a8.png)

17. Buat file bernama dockerfile

        FROM node:10
        WORKDIR /app
        COPY . .
        RUN npm install
        RUN npm install sequelize-cli
        RUN wget https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh
        RUN chmod +x wait-for-it.sh
        RUN echo '#!/bin/bash' > start-server.sh
        RUN echo 'npx sequelize-cli db:migrate' >> start-server.sh
        RUN echo 'npm start' >> start-server.sh
        RUN chmod +x start-server.sh
        EXPOSE 5000
        CMD ["npm", "start"]

    ![image](https://user-images.githubusercontent.com/40049149/189877412-13f0efc7-2962-448b-96a5-7978460e3b12.png)

18. Sekarang buat file docker-compose.yaml dengan isi berikut:


        version: '3.7'
        services:
          backend:
            build: .
            depends_on:
              - 'db'
            ports:
              - '5000:5000'
            expose:
              - '5000'
            command: ./wait-for-it.sh db:3306 -s -t 0 -- ./start-server.sh
          db:
            image: 'mysql:latest'
            restart: 'unless-stopped'
            environment:
              - 'MYSQL_DATABASE:housy'
              - 'MYSQL_USER:menther'
              - 'MYSQL_PASSWORD:Bootcamp13!@'
              - 'MYSQL_ROOT_PASSWORD:Bootcamp13'
            ports:
              - '3306:3306'
            expose:
              - '3306'
            volumes:
              - 'database:/var/lib/mysql'


    ![image](https://user-images.githubusercontent.com/40049149/189880454-40242e97-010a-42db-8a06-9c73bb436da1.png)

19. Build dan Jalankan compose secara daemon dengan:

        docker-compose up -d

    ![image](https://user-images.githubusercontent.com/40049149/189887940-52c598af-3787-4c8a-a1d6-80d0725a4557.png)

    ![image](https://user-images.githubusercontent.com/40049149/189888556-5a1e832f-efdf-42fe-a40b-91ffb06ad18d.png)






