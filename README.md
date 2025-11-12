# OHConsol
Web TFTP Server / Serial Console - OH?!?


Tired of figuring out how your going to tftp that new firmware image to a device? Tired of misspelling files names or losing track of what that image.out file really is? Tired of boucning from some tftp server to some other serial console?


Simply upload your file with a description (optional) and activate it. That file is now ready for a tftp download as image.out. 

But wait?!? I need console access... Keep reading
![alt text](https://github.com/2jgit/OHConsol/blob/main/files/TFTP%20Upload.jpg)

Pass your USB Serial Cable into OHConsol - /dev/ttyUSB0 (You'll need to confirm this on your host via dmesg and update docker-compose.yml accordingly)
Now you have a web serial terminal.
![alt text](https://github.com/2jgit/OHConsol/blob/main/files/Serial%20Terminal.jpg)


Adjust below docker compose file as needed. 

## If you wish to only use the tftp web server and do not have a usb serial device to passthrough ##
## Comment out the device section and set SERIAL_ENABLED: false in the env variables for ohconsol_web ##


Once delpoyed 
 - access web console via http://localhost:8080 (http://X.X.X.X:8080)
 - TFTP Access via udp/69

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
    image: 2jdock/2jpub:ohconsol
    container_name: ohconsol_web
    depends_on:
      - ohconsol_db
    ports:
      - "8080:80"        # Access webapp on host:8080
      - "8081:8081"     # required for serial websocket
      - "69:69/udp"    # TFTP UDP port forwarded to host

    ### Serial console is enabled by default. If you wish to only run tftp web service
    ### comment out "devices:" and " - "/dev/ttyUSB0:/dev/ttyUSB0" "
    ### and set SERIAL_ENABLED: false in the env below
    devices:
      - "/dev/ttyUSB0:/dev/ttyUSB0" # change host side to your USB device mapping
    privileged: true   # required for serial access
    environment:
      DB_HOST: ohconsol_db
      DB_NAME: fileapp
      DB_USER: fileapp
      DB_PASS: fileapp_pw
      SERIAL_ENABLED: true
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
