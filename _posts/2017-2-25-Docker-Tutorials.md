<!-- # _Docker_ -->
This is a tutorial about base operation of Docker.

## Common command:
- List all installed images.<br>
    ```
    docker images
    ```

- List all containers.<br>
    ```
    docker ps -a
    ```

- Stop all containers.<br>
    ```
    docker stop $(docker ps -a -q)
    ```
    -q(--quiet), Only display numeric IDs.

- Remove a container.<br>
    ```
    docker rm <contrain name>
    ```

- Remove all containers.<br>
    ```
    docker rm $(docker -a -q)
    ```

- Remove a image.<br>
    ```
    docker rmi <image name and tag>
    ```

- Run a container which is already exiting.<br>
    ```
    docker start <container_id or name>
    docker exec -ti <container_id or name> /bin/bash
    ```

- Copy file from host into container in shell (From [stackoverflow][ref_1]).<br>
    ```
    docker cp <file name> container:<path>
    ```

- Convert container into image.<br>
    ```
    docker commit <container name/id> <image name>
    ```
    It is used to build new image from container.

- Save docker image into host disk.<br>
    ```
    docker save -o <path you want to save> <image name and tag>
    ```
    often save docker image as .tar

- Load docker image from disk.<br>
    ```
    docker load -i <image file path>
    ```

## Valuble Docker Image
- [dl-docker](https://github.com/floydhub/dl-docker):
    all-in-one docker image for deep learning.


## Problem and Solution
- How to share fold betweent host and container on Docker for Windows? [(ref)][ref_2]
    - Open Docker for Windows
    - Set `Shared Drives`
    - run docker contrainer with `-v [fold path on host]:[fold path on contrainer]`

- How to use jupyter notebook in docker? localhost:8888 not work? [(ref)][ref_3]
    - _Solution:_ The ip of container in docker is 0.0.0.0, but default ip address in jupyter is 127.0.0.1. So we should change jupyter notebook ip if we want to use it on our host computer. Input `jupyter note --ip=0.0.0.0` in your docker container and then open localhost:8888 in your browser, and see it will work ok.

- On docker for Windows, how host computer powershell direct ping to container(container ip is 172.17.0.1)?
    run docker container with `--net=<net bridge>`, set a net bridge for container.

- When delete image, error `Can’t delete image with children
` occur.  
    Use `docker inspect --format='{{.Id}} {{.Parent}}' $(docker images --filter since=<image_id> -q)`. And then delete sub_image using `docker rmi {sub_image_id}`

- Using python in Docker Linex container, has error like `UnicodeEncodeError: 'ascii' codec can't encode character '\u22f1' in position 242`
    - Solution:
        - [解决Python3下打印utf-8字符串出现UnicodeEncodeError的问题](https://www.binss.me/blog/solve-problem-of-python3-raise-unicodeencodeerror-when-print-utf8-string/)

        ```python
        >>> import sys
        >>> sys.stdout
        <_io.TextIOWrapper name='' mode='w' encoding='ANSI_X3.4-1968'>
        ```

        Change output encode as utf-8
        ```python
        >>> import sys
        >>> import io
        sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
        ```

## Tips
- Useful command in ubnutu image:
    - Change ubuntu download source, using `sed`<br>
        ```
        sed -i s@<needed replace content>@<replace content>@g <file path>
        ```

        E.g.
        ```shell
        sed -i s@http://archive.ubuntu.com/ubuntu/@http://mirrors.tuna.tsinghua.edu.cn/ubuntu/@g /etc/apt/sources.list
        ```

        After you change the source list, you need to update it to let it work via `sudo apt-get update`

- How run linux gui in docker container on docker for windows?
    - *Solustion*
        1. Install **Cygwin** with **Cygwin/x** on your computer.

        2. In cygwin terminal, run
            ```shell
            export DISPLAY=<your-machine-ip>:0.0
            startxwin -- -listen tcp &
            xhost + <your computer ip>
            ```

        3. In your powershell, run
            ```
            docker run --it -e DISPLAY=<your computer ip>:0.0 <image> /bin/bash
            ```
    - *Problem*
        - Error: `xhost:  unable to open display`(\*[ref](https://forums.freebsd.org/threads/50613/))  
            Use `rm ~/.Xauthority`, then try again previous steps.

    - References
        - [Blog: Running a GUI application in a Docker container](https://linuxmeerkat.wordpress.com/2014/10/17/running-a-gui-application-in-a-docker-container/)

        - [Running Linux GUI Apps in Windows using Docker](https://manomarks.net/2015/12/03/docker-gui-windows.html)

        - [Stackoverflow: How to run GUI application from linux container in Window using Docker?](http://stackoverflow.com/questions/29844237/how-to-run-gui-application-from-linux-container-in-window-using-docker)

        - [Linux: Running GUI apps with Docker](http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/)

- Using **nvidia-docker** run cuda in docker container.
    - Install **nvidia-docker** from [here]()
    - Problems and Solutions
        - `Error: Could not load UVM kernel module. Is nvidia-modprobe installed?`
            - Install `nvidia-modprobe`([\*ref](https://askubuntu.com/questions/841824/how-to-install-nvidia-modprobe))<br>
            ```shell
            sudo apt-add-repository multiverse
            sudo apt update
            sudo apt install nvidia-modprobe
            ```

        - `Error response from daemon: create nvidia_driver_352.63: create nvidia_driver_352.63: Error looking up volume plugin nvidia-docker: plugin not found.`
            1. Check the status of the plugin using via `service nvidia-docker status` or `systemctl status nvidia-docker`
            2. run `sudo nvidia-docker-plugin` in shell
            3. restart docker `sudo restart docker`
            4. logout
            5. test `nvidia-docker run --rm nvidia/cuda nvidia-smi` again

## References
- [Docker Toolbox, Docker Machine, Docker Compose, Docker WHAT!?](https://nickjanetakis.com/blog/docker-toolbox-docker-machine-docker-compose-docker-wtf)
- [Docker Explained](https://www.digitalocean.com/community/tutorials/docker-explained-using-dockerfiles-to-automate-building-of-images)

[ref_1]:http://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container
[ref_2]:https://rominirani.com/docker-on-windows-mounting-host-directories-d96f3f056a2c#.8tny4uf9o
[ref_3]:https://github.com/gopherds/gophernotes/issues/6
