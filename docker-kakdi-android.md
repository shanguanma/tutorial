# Install and use 

Refrence: https://github.com/jcsilva/docker-kaldi-android

# Install docker

Please, refer to https://docs.docker.com/engine/installation/.

# Get the image

- Pull the image from Docker Hub:

  `$ docker pull jcsilva/docker-kaldi-android`

- Or build your own image (requires git):

  `$ git clone https://github.com/jcsilva/docker-kaldi-android`

  `$ cd docker-kaldi-android`

  `$ docker build -t jcsilva/docker-kaldi-android:latest .`

# How to use

- First of all, clone Kaldi repository to somewhere in your machine.

  `$ git clone https://github.com/kaldi-asr/kaldi`
- Run a container, mounting the Kaldi repository you just cloned to /opt/kaldi:

  `$ docker run -v /home/test/kaldi:/opt/kaldi jcsilva/docker-kaldi-android:latest`
- In the above example, it was assumed the Kaldi repository was cloned to /home/test/kaldi. 
  If you cloned it to somewhere else, you must change it accordingly.

- When docker run finishes, all the compiled Android binaries will be located in /home/test/kaldi/src subdirectories.
