# Docker and Kubernetes Demo

In this directory you'll find a Python script, a Dockerfile, and a job submission yaml file, job-runner.sh. This is all you need to create a simple docker image and run the container on kubernetes. Lets dive in!

## Part 1. Build a simple docker image on your local machine

Note: everything here is for mac. It might be slightly different for windows.

1. Create a docker image on your machine using the dockerfile. We are going to call it demo:latest.
```
$ cd <current directory>
$ docker build -t demo:latest .
```

2. Run the docker on your local machine
```
$ docker run demo
```
Did it do what you wanted? Did you expect it to do anything? Try the interactive mode:
```
$ docker run -it demo
```

3. explore:
   - make it start from the terminal. once you do, examine the folder structure inside.
   - make it automatically run the hello.py script.
   - make your own, add libraries, build, and run again.
   - How can you keep track of changes?
   - how do you resrrect old images?
      - make use of hpcharbor and dockerhub as repositories.
      - also, you can always save them out to local tar files using `docker image save ..`

4. what are some other commands you could use? when would you use them?  

## Part 2. Upload image to HPCharbor 

Before you proceed, we are going to make some changes. 

1. rebuild the image using `--platform=linux/x86_64` argument
```
$ docker build --platform linux/x86_64 -t demo:latest .
```

THIS IS AN ABSOLUTE NECESSARY STEP FOR ANYTHING TO RUN ON THE KUBERNETES SIDE.

3. Tag it for HPC harbor upload and Push it (https://hpcharbor.mdanderson.edu/harbor/projects). 
   
```
$ docker tag demo:latest hpcharbor.mdanderson.edu/<your_folder>/demo:latest
$ docker push hpcharbor.mdanderson.edu/<your_folder>/demo:latest
```

## Part 3. Run your container with kubernetes

I am assuming you have a kubernetes account, your HOME directory has been mounted and that they have given you a folder with templates.
Take a look at `job.hello.gpu.yaml` as an example.

I highly recommend installing OpenLens on your machine. It helps a lot with monitoring job progress (Talk to Ping about instructions and setting it up)

1. create your own job template. 
   - Find the template provided to you in your home directory. Mine was in /rsrch4/home/plm/sranjbar/k8s-templates
   - There are 2 templates, one for gpu and one for cpu. I will be using the gpu one here.
   - duplicate your gpu template inside the code directory, rename it something meaningful.

2. change `metadata.name, containers.image, containers.command`. Follow what I have inside `job.hello.gpu.yaml`.

Ok, you now have a job template that looks for your image on HPCharbor, and calls `python hello.py` when running it.

3. ssh into your seadragon account and cd inside this directory. 

4. lets run the job. you can do it 2 ways, using job-runner.sh or using the kubectl apply command.

```
job-runner.sh job.hello.gpu.yaml
```
5. check progress by looking at your job status in OpenLens or you can wait and see what happens. Once the job is done, you will see a `logs` folder and a `done` folder appear in your directory. Check out the contents of the log folder for details.


## Trouble shooting

- If in Part 3 -> step 4 the log file says ```exec /usr/local/bin/python: exec format error```, you haven't paid enough attention and need to go back to Part2 -> step 1.

