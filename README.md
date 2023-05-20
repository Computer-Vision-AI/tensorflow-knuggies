# tensorflow-knuggies
The repo contains examples of tesorflow 2.0 and keras.

# Run Tensorflow in a Docker Container
To run Tensorflow in a Docker Container, we’ll run the following command from the WSL terminal:

```zsh
docker run -it --rm -p 8888:8888 --gpus all tensorflow/tensorflow:latest-gpu-jupyter
```

The first time running the command, it will download the Tensorflow container from dockerhub. Once in downloads and runs, click the link to open the Jupyter Notebook server:

```zsh
http://127.0.0.1:8888/?token=some-gibberish-yours-will-be-different
```

From here, we can create a new notebook and run the following Python code to confirm that we can see the GPU from the notebook running in the Tensorflow Docker Container.

```jupyter
import tensorflow as tf
tf.config.list_physical_devices()
```
This is a great start, but if we stop and restart the container, all of our work is gone. Let’s fix that.

# Improve the Environment with docker-compose
Now that we have confirmed that the Docker container running Tensorflow has access to the GPU, let’s improve the development environment by creating an easy way to restart our environment __and save our work__. For this, we’ll need to create a few files. The three files named “requirements.txt”, “Dockerfile” and “docker-compose.yaml” have contents as shown below:

requirements.txt

```zsh
jupyterlab
pandas
matplotlib
```

Dockerfile:

```bash
FROM tensorflow/tensorflow:latest-gpu
WORKDIR /tf-knugs  # This specifies the directory to work
RUN pip install --upgrade pip
RUN pip install --upgrade -r requirements.txt
EXPOSE 8888
ENTRYPOINT ["jupyter", "lab","--ip=0.0.0.0","--allow-root","--no-browser"]
```
docker-compose.yaml

```basg
services:
  jupyter-lab:
    build: .
    ports:
      - "8888:8888"
    volumes:
      - ./tf-knugs:/tf-knugs
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

Note that the directory “tf-knugs” is used for mapping a volume from the WSL system to the container. These names can be changed to any valid directories. The key is that the WSL directory is on the left side of the “:” and the container directory is on the right side. The directory on the right side should match the “WORKDIR” specified in the Dockerfile.

Once you have these three files saved in the same directory, we can run the new and improved environment by running the following commands to build the container then compose the environment: 

```zsh
docker build .
docker compose up
```

Once these steps complete, we’ll see a familiar link to Jupyter Notebooks, but now it opens the newer Jupyter Lab environment that we installed from the “requirements.txt” file.

To stop the environment, go to the WSL terminal used to run “docker-compose up” and press ctrl-C. Alternatively, open a new terminal in the same directory and run:

```zsh
docker compose down
```
