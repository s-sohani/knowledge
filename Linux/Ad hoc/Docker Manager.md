```
آقا این portainer رو دریابین. خیلی چیز خوبیه. یه واسط کاربری خیلی ساده و کاربردی برای مدیریت container های داکر هست.  
  
docker run --name portainer -d -p 5010:9000 -p 5011:8000 -p 5012:9443 --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /data/portainer:/data portainer/portainer-ce:latest
```


## For mirror docker in iranserver
```
{
  "registry-mirrors": ["https://docker.iranserver.com"]
}
```