# deploy-angular-ssr-to-forge
Instructions for deploying angular universal/ssr to laravel forge

## 1. Provision a web server on Forge with php and nginx
## 2. Install nvm on server (for Angular v.16 use node 18.10.0) 
## 3. Install the Angular ssr repository onto the server
## 4. In the nginx file:
Replace this:
```
   location / {
         try_files $uri $uri/ /index.php?$query_string;
     }
```    
With this: 
```
    location / {    
    proxy_pass http://localhost:4000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    }
```

## 5. Install @angular/cli on the server
npm i -g @angular/cli@latest

## 6. Add a daemon with these settings

command: ``` node dist/YOUR_APP_NAME/server/main.js ```\
directory: ``` home/forge/default ```

## 7. Edit the deploy script to:
```
cd /home/forge/default

git fetch origin
git checkout $FORGE_SITE_BRANCH
git pull origin $FORGE_SITE_BRANCH

( flock -w 10 9 || exit 1
    echo 'Restarting FPM...'; sudo -S service $FORGE_PHP_FPM reload ) 9>/tmp/fpmlock

npm install

ng build && ng run YOUR_APP_NAME:server

echo "" | sudo -S supervisorctl restart all
```

## 8. set up cron job to check the status of node process: 
```
*/5 * * * * bash default/dist/YOUR_APP_NAME/server/check_node_process.sh
```
## 9. Configure supervisor.sock permissions
- make forge user group an owner
```
sudo chown root:forge /var/run/supervisor.sock
sudo chmod 0770 /var/run/supervisor.sock
```
- edit supervisord.conf
```
sudo nano /etc/supervisor/supervisord.conf
```
```
[unix_http_server]
file=/var/run/supervisor.sock
chmod=0770
chown=root:supervisorusers
```
- restart supervisor
```
sudo service supervisor restart
```
