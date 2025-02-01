# SetupDroneCi
There are several components that come into play while setting up drone-ci. The technology is container based so, a docker environment is required, while we write the steps in the pipeline, images to set the environment are defined and run in the container. Drone has two components, drone-ci and drone-runner, the drone runner with the help of github oauth helps to set hooks in particular repo and triggers can be set. 

## Setting up Github OAuth
It is required to safely authenticate with our organization's GitHub and setup hooks.

To create a Github OAuth application: 
- Navigate to your GitHub Profile
- On top right you can find your profile image, click there.
- You can find ```settings``` option there, click it.
- At the left navigation bar of your page, nativage to ```Developer settings```.
- Now you can find the ```Oauth Apps``` options.
- Click on ```New OAuth App```.
- Fill in the details.

 <img width="1440" alt="Screen Shot 2025-02-01 at 12 01 12 PM" src="https://github.com/user-attachments/assets/27f1177d-ef86-4fce-9b15-850d3f6aad96" />

The home url should be the url of the host and port of where the drone app is running and the return url must be the same as homeurl /login.
**Note**: Drone requires publicly hosted instances to initiate the webhooks, so please use a public ipaddress or use ngrok as I have done.
**Also**: Keep in mind if the terminal running ngrok gets terminated, all of this will not work as ngrok on free will refresh your ip address.

- After the auth application is created you will get the client ID and Client secret, at first client secret needs to be generated. After generation please copy it in a secure environment because you will not be able to access it yourself after the tab is closed.

## SetUp Drone and Drone_Runner

After all copying all the secrets and IDs, we can finally write a docker compose file to run drone-ci and drone-runner.

First we need to set up all the environment variables either indicidually or in the .env file. I am doing it in .env file which is easier to manage.

content in .env
```
# Server configuration
DRONE_SERVER_HOST=<ngrok hostname>
DRONE_SERVER_PROTO=http
DRONE_GITHUB_CLIENT_ID=<github client ID>
DRONE_GITHUB_CLIENT_SECRET=<github secret>
DRONE_RPC_SECRET=<Drone secret>
DRONE_TLS_AUTOCERT=false
DRONE_USER_CREATE=username:<github_username>,admin:true

# Runners configuration
DRONE_RPC_HOST=<ngrok hostname>
DRONE_RPC_PROTO=http
DRONE_RUNNER_NAME="Drone.io_runner"
```

everything is self-explanatory except for DRONE_RPC_SECRET=<Drone secret>, this can be generated using ```openssh rand -hex 16``` in shell.

Now let's write the docker-compose.yml file

```
version: '3.8'
services:
  drone:
    image: 'drone/drone:2'
    container_name: drone
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "./volumes/drone:/data"
      - '/etc/localtime:/etc/localtime:ro'
    restart: always
    networks:
      - dronenet
    ports:
      - 80:80
    environment:
      - DRONE_SERVER_HOST=${DRONE_SERVER_HOST}
      - DRONE_SERVER_PROTO=${DRONE_SERVER_PROTO}
      - DRONE_GITHUB_CLIENT_ID=${DRONE_GITHUB_CLIENT_ID}
      - DRONE_GITHUB_CLIENT_SECRET=${DRONE_GITHUB_CLIENT_SECRET}
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
      - DRONE_TLS_AUTOCERT=${DRONE_TLS_AUTOCERT}
      - DRONE_USER_CREATE=${DRONE_USER_CREATE}
  drone_runner:
    image: 'drone/drone-runner-docker:1'
    container_name: runner
    command: agent
    depends_on:
      - drone
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - '/etc/localtime:/etc/localtime:ro'
    networks:
      - dronenet
    environment:
      - DRONE_RPC_HOST=${DRONE_RPC_HOST}
      - DRONE_RPC_PROTO=${DRONE_RPC_PROTO}
      - DRONE_RUNNER_NAME=${DRONE_RUNNER_NAME}
      - DRONE_RUNNER_CAPACITY=${DRONE_RUNNER_CAPACITY}
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
networks:
  dronenet:
```
Now just run ```docker-compose up -d``` 

to run drone from cli from your host computer
pass
```
export DRONE_SERVER=http://drone.mycompany.com
```
```
export DRONE_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```
That is how we can setup and run drone-ci, the actual running commands are not mentioned here just the setup part only.
