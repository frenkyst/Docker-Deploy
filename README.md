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

13. Buat file mysql.yaml di home atau mana terserah

        version: '3.7'
        services:
           mysql:
             image: mysql:8
             ports:
              - 3306:3306
             volumes:
              - ~/apps/mysql:/var/lib/mysql
             environment:
              - MYSQL_ROOT_PASSWORD=Bootcamp13
              - MYSQL_PASSWORD=Bootcamp13!@
              - MYSQL_USER=menther
              - MYSQL_DATABASE=housy

    ![image](https://user-images.githubusercontent.com/40049149/190401655-938ac0b0-419b-4317-ab4a-e81b4835587f.png)

14. Jalakan dengan perintah berikut

        docker compose -f mysql.yaml up -d

    ![image](https://user-images.githubusercontent.com/40049149/190402843-6b2804fe-4fb7-4fa9-914d-b12b81c7af86.png)

15. Buat file dockerfile di directory frontEnd

        FROM node:10-alpine
        WORKDIR /usr/app
        COPY . .
        RUN npm install
        EXPOSE 3000
        CMD [ "npm", "start" ]

    ![image](https://user-images.githubusercontent.com/40049149/189845897-d3541a0a-00d4-4b99-9af0-280f3f49f42c.png)

16. Buat file docker-compose.yaml untuk frontend dengan isi:

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

17. Build dan Jalankan compose secara daemon

        docker-compose up -d
        
    ![Screenshot from 2022-09-13 15-19-37](https://user-images.githubusercontent.com/40049149/189850145-bd946097-bcba-4180-bf08-d8da1a09bf6f.png)

18. Ke directory BackEnd edit file config.json

        nano config/config.json

    ![image](https://user-images.githubusercontent.com/40049149/189876567-47a53386-b071-4d77-8a06-c1f8f212925e.png)

    ![Screenshot from 2022-09-13 17-16-18](https://user-images.githubusercontent.com/40049149/189876748-f43a4018-7927-46cc-b8e0-e6397c63a3a8.png)

19. Buat file bernama dockerfile

        FROM node:10-alpine
        WORKDIR /usr/app
        COPY . .
        RUN npm install
        RUN npm install sequelize-cli -g
        RUN npx sequelize db:migrate
        EXPOSE 5000
        CMD ["npm", "start"]

    ![image](https://user-images.githubusercontent.com/40049149/190343602-f3ea2933-12d7-4d5b-ae67-93cdfc080219.png)

20. Sekarang buat file docker-compose.yaml dengan isi berikut:


        version: '3.7'
        services:
         backend:
           build: .
           container_name: backend1
           stdin_open: true
           ports:
            - '5000:5000'


    ![image](https://user-images.githubusercontent.com/40049149/190344022-33410228-36af-4642-b787-abf909fb0cd6.png)

21. Build dan Jalankan compose secara daemon dengan:

        docker-compose up -d

    ![image](https://user-images.githubusercontent.com/40049149/189887940-52c598af-3787-4c8a-a1d6-80d0725a4557.png)

22. Cek apakah aplikasi sudah jalan

    ![image](https://user-images.githubusercontent.com/40049149/190405745-ca6ec04e-8f4d-41bd-9062-bbcf49f84bb1.png)






