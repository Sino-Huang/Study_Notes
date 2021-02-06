# Docker basic 

## setting docker without sudo 

To create the `docker` group and add your user:

1. Create the `docker` group.

   ```
   $ sudo groupadd docker
   ```

2. Add your user to the `docker` group.

   ```
   $ sudo usermod -aG docker $USER
   ```

3. Log out and log back in so that your group membership is re-evaluated.



## check docker image info 

### docker inspect {image}

```
docker inspect scrapinghub/splash | less
```

### 

##### Less Command – Search Navigation

Once you’ve opened a log file (or any file) using ***less file-name\***, use the following keys to search. Please note that the match will be highlighted automatically by default.

##### Forward Search

-  / – search for a pattern which will take you to the next occurrence.
-  n – for next match in forward
-  N – for previous match in backward

##### Backward Search

-  ? – search for a pattern which will take you to the previous occurrence.
-  n – for next match in backward direction
-  N – for previous match in forward direction



You can find Port 

            "ExposedPorts": {
                "8050/tcp": {}
            },


## docker run 

1. -d 
   1. means detach, you run it in the background 

```
# settings when docker run

docker run -d --name {name} {container}
```



## docker stats

show the stats of the current running docker 



```
# show all containers 
docker stats -a  
docker ps -a
```



## docker stop / rm 

remove docker container 

```
# -f force to remove 
docker rm -f {container name or id}
```





## docker images

check the local docker images 



