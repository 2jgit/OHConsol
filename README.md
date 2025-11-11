# OHConsol
Web TFTP Server / Serial Console - OH?!?


Tired of figuring out how your going to tftp that new firmware image to a device? Tired of misspelling files names or losing track of what that image.out file realy is? Tired of boucning from tftp server to console?


Simply upload your file with a description (optional) and activate it. That file is now ready for a tftp download as image.out. 

But wait?!? I need console access... Keep reading
![alt text](https://github.com/2jgit/OHConsol/blob/main/files/TFTP%20Upload.jpg)

Pass your USB Serial Cable into fortifirm - /dev/ttyUSB0 (You'll need to confirm this on your host via dmesg and update docker-compose.yml accordingly)
Now you have a web serial terminal.
![alt text](https://github.com/2jgit/OHConsol/blob/main/files/Serial%20Terminal.jpg)


Adjust below docker compose file as needed. 

## Current Version fails if USB device passthrough is not configured ##
## V2 will allow option to disable for TFTP Service only ##

```
services:
  ohconsol_db:
    image: mariadb:11
    container_name: ohconsol_db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: example_root_pw
      MYSQL_DATABASE: fileapp
      MYSQL_USER: fileapp
      MYSQL_PASSWORD: fileapp_pw
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - host

  ohconsol_web:
    image: 2jdock/2jpub:ohconsol:latest
    container_name: ohconsol_web
    depends_on:
      - ohconsol_db
    ports:
      - "8080:80"        # Access webapp on host:8080
      - "8081:8081"     # required for serial websocket
      - "69:69/udp"    # TFTP UDP port forwarded to host
    devices:
      - "/dev/ttyUSB0:/dev/ttyUSB0" # change host side to your USB device mapping
    privileged: true   # required for serial access
    environment:
      DB_HOST: fortifirm_db
      DB_NAME: fileapp
      DB_USER: fileapp
      DB_PASS: fileapp_pw
    volumes:
      - uploads:/var/www/html/uploads
      - tftp:/mnt/tftp
    restart: unless-stopped
    networks:
      - host
    cap_add:
      - NET_BIND_SERVICE

volumes:
  db_data:
  uploads:
  tftp:

networks:
  host:
    driver: bridge
```
