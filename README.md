# mozdefDockerMeteor
Mozdef Meteor UI running in docker

Dockerfile and supporting scripts (based on https://github.com/abernix/spaceglue) to allow a docker container to be built
statically (not dynamic updating on a prod server) on a dev workstation and deployed to prod running mozdef's meteor-based UI.

General steps:
#build mozdefUI
    #rm all previous docker containers/images
    docker ps -a
    <list>
    docker rm <imageid>
    
    docker images
    <list>
    docker rmi $(docker images -qf "dangling=true")
    
    
    #bundle existing mozdef UI project:
    <cd to dir where you developed meteor UI>
    meteor build --architecture=os.linux.x86_64 ./
    cp <dirname.tar.gz> <dockerdir where you house the Dockerfile to build an image>
    
    #build by unpacking bundle and installing all npm modules
    docker build --tag mozdefui/base --label mozdefui --build-arg INSTALL_NPM=1 .
    
    #save docker image:
    docker save mozdefui/base | gzip > ../mozdefuibase.tar.gz
    
    #scp to server:
    scp ../mozdefuibase.tar.gz <your target server here>:
    
    #import on server:
    mv ~/mozdefuibase.tar.gz /path/to/your/dockercontainers
    gunzip -c mozdefuibase.tar.gz| docker load
    
    #debug run on server talking to local mongo on 127.0.0.1:
    docker run -d --net=host -p 3000:3000 -e PORT=3000 -e MONGO_URL=mongodb://127.0.0.1:3002/meteor -it --entrypoint /bin/bash mozdefui/base
    docker attach <containerID>
    node bundle/main.js
    
    #prod run on server <no attach>
    docker run -d --net=host -p 3000:3000 -e PORT=3000 -e MONGO_URL=mongodb://127.0.0.1:3002/meteor mozdefui/base node bundle/main.js
    
    