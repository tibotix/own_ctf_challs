This Repository includes some of my personally created CTF Challenges mainly for own educationally purposes.

Each Challenge includes three folders.

1. `build` : this directory includes all files necessary to build the challenge locally
2. `deploy` : this directory includes all files necessary to deploy the challenge locally on docker
3. `solution` : this directory includes the intended solution and a WriteUp (ignore/remove this folder if you want to solve it on your own)

To deploy a challenge you first have to build the docker image:

```bash
sudo docker build -t <challenge name> deploy/Dockerfile
sudo docker run -d --rm -p1024:1024 <challenge name>
```

