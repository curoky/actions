# Login github action runner

## With Tmate

A good idea is use [debugging-with-tmate](https://github.com/marketplace/actions/debugging-with-tmate).

1. write action config

   add follow lines to action config files.

   ```yaml
   name: connect with tmate
   on:
     push:
   jobs:
     connect_ubuntu:
       runs-on: ubuntu-20.04 # 16/18/20 or macos-10.15 or windows-2019
       steps:
         - uses: actions/checkout@v2
         - name: Setup tmate session
           uses: mxschmitt/action-tmate@v2
   ```

2. login runner

   To get the connection string, just open the Checks tab in your Pull Request and scroll to the bottom. There you can connect either directly per SSH or via a web based terminal.

   ![tmate login message](./images/tmate.png 'tmate login message')

3. stop and continue

   Just create a file `/continue`.

   ```bash
   sudo touch /continue
   ```

## With [ngrok](https://ngrok.com/)

1. signup ngrok [account](https://dashboard.ngrok.com/signup)

2. write action config

   ```yaml
   name: connect with ngrok

   on:
     push:

   jobs:
     connect_ubuntu:
       runs-on: ubuntu-20.04 # 16/18/20 or macos-10.15 or windows-2019
       steps:
         - name: install ngrok
           run: |
             wget -q https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
             unzip ngrok-stable-linux-amd64.zip
             chmod +x ./ngrok

         - name: run ngrok
           run: |
             echo -e "123456\n123456" | sudo passwd "$USER"
             ./ngrok authtoken "1gBog0sY5zyuE0EJKj76qynOXC8_4NmBLqhyJL9Fo6i5D6jsU"
             ./ngrok tcp 22 --log ".ngrok.log" &
             sleep 10
             echo "To connect: $(grep -o -E "tcp://(.+)" < .ngrok.log | sed "s/tcp:\/\//ssh $USER@/" | sed "s/:/ -p /")"

         - name: don't kill instace
           run: sleep 1h # Prevent to killing instance after failure
   ```

3. login runner

   ![ngrok login message](./images/ngrok.png 'ngrok login message')

## Reference

- <https://dev.to/retyui/how-debugging-github-actions-with-ssh-273n>
