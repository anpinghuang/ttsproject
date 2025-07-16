# ttsproject

digital ocean

ubuntu 24.10 x64

basic, premium intel NVME SSD, $32/month

using ssh, public_key_api

1 click install docker (marketplace) on ubuntu




# update packages
apt update & apt upgrade -y

# check docker

docker --version

// response
root@tts:~# docker --version
Docker version 28.3.2, build 578ccf6

# insrtall docker compose

apt install docker-compose -y

docker-compose --version


# create a docker compose file
nano docker-compose.yml

// paste this
version: '3'
services:
  tts:
    image: ghcr.io/coqui-ai/tts-cpu:main
    restart: unless-stopped
    ports:
      - "5002:5002"
    entrypoint: >
      python3 TTS/server/server.py
      --model_name tts_models/en/ljspeech/tacotron2-DDC
      --port 5002

# start TTS service

docker-compose pull


# start service
docker-compose up -d

# check container is running

docker ps

// result (should say "up")

CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                                         NAMES
7bc5a62daea5   ghcr.io/coqui-ai/tts-cpu:main   "python3 TTS/server/…"   15 seconds ago   Up 14 seconds   0.0.0.0:5002->5002/tcp, [::]:5002->5002/tcp   tts_tts_1
root@tts:~/tts#


# add domain to digital ocean, add A record pointing to droplet and connect with namecheap

// e.g .
Type: A
Host: tts
Value: your droplet public IP
TTL: Automatic

// mine domain is tts.fakereal.us


# install nginx and https
apt install nginx -y

# configure nginx
nano /etc/nginx/sites-available/default

// ctrl+K to delete line by line, then replace entire with:

server {
    listen 80;
    server_name tts.fakereal.us;

    location / {
        proxy_pass http://127.0.0.1:5002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}


# test config

nginx -t

// result

root@tts:~/tts# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful


# reload nginx

systemctl reload nginx



----------------- still on this step (start below after namecheap updates)

# get back to folder 
cd ~/tts

# check if domain points to right IP
nslookup tts.fakereal.us

# check droplet firewall
ufw status
ufw allow 80
ufw allow 443
ufw reload

// result 
22/tcp                     ALLOW       Anywhere


# install certbot (SSL) 

apt install certbot python3-certbot-nginx -y

# enable https

certbot --nginx -d tts.fakereal.us



# test it locally!

create payload.json
{
  "text": "Hello from my PC!"
}

run curl
curl.exe -v -X POST "https://tts.fakereal.us/api/tts" -H "Content-Type: application/json" -d "@payload.json" --output output.wav



# debugging
## show logs
docker logs tts_tts_1 2>&1 | tail -n 100

## use from local PC 
// tip: only enter words for the input, not URLs because it will start hallucinating if you give it some strange symbols to read. 

curl.exe -X POST "https://tts.fakereal.us/api/tts" -H "text: your message here" --output output.wav



## chatgpt for help
https://chatgpt.com/share/68780814-9cd8-800e-a3ef-0bb4d588c556


## to update with new model
nano docker-compose.yml

model options
| Model Name                                       | Language                | Description                                                           |
| ------------------------------------------------ | ----------------------- | --------------------------------------------------------------------- |
| `tts_models/en/ljspeech/tacotron2-DDC`           | English                 | ⭐ Most stable. Medium quality, single speaker.                        |
| `tts_models/en/ljspeech/tacotron2-DDC_ph`        | English                 | Tacotron2 with phoneme support for better pronunciation.              |
| `tts_models/en/ljspeech/glow-tts`                | English                 | Fast and high-quality Glow-TTS model.                                 |
| `tts_models/en/ljspeech/fast_pitch`              | English                 | Fast and modern, with pitch control.                                  |
| `tts_models/en/vctk/vits`                        | English (multi-speaker) | Supports speaker embeddings (VCTK dataset, many voices).              |
| `tts_models/multilingual/multi-dataset/your_tts` | Multilingual            | Multi-speaker, multi-lingual, supports voice cloning. Very versatile. |
| `tts_models/en/jenny/jenny`                      | English                 | 🎙️ Trained on female voice “Jenny.” High quality, emotional range.   |
| `tts_models/fr/mai/tacotron2-DDC`                | French                  | High quality French voice.                                            |
| `tts_models/de/thorsten/tacotron2-DCA`           | German                  | German voice model.                                                   |


Save and restart the container:

docker compose down


docker compose up -d

