# Docker Deep Dive - Chatper 8 Notes

Containerizing an app

* Dockerizing, or containerizing
* Simple linux app in play-with-docker
* TLDR
  - make apps simple to build, ship, run
  - process: 
    1. your code
    1. Create recipe - Dockerfile - describes your app, its dependencies, and how to run
    1. Build Dockerfile with with `docker image build`
* The deep dive
  - Containerizing sinlge-container app - get code, inspect Dockerfile, containerize app, run app, test app, move to production
      1. Dockerfile 
        - build context - root of your app
        - Bridges gaps b/t dev and ops as a form of documentation
        - start with alpine linux, add meta via `LABEL`, install dependencies (ie node), get container at app code, set working dir in container, restore app dependencies, doc app's port, set default to app run
        - Starts with **FROM**, which is the base layer. Each line is a layer on top.
      1. Building via `docker image build -t web:latest .`
        - Check exists, `docker image ls`
        - Inspect, `docker inpsect web:latest`
      1. Push image - store in registry, Docker Hub by default
        - `docker login` 
        - Specify registry, repo, tag:  `docker image tag web:latest nigelpoulton/web:latest`
        - Push: `docker image push nigelpoulton/web:latest`
      1. Run `docker container run -d --name c1 -p 80:8080 web:latest`
    - Comments are #, INSTRUCTION argument, metadata doesn't create layers (ie LABEL)
    - `docker image history web:latest`
  - Production with multi-stage builds
    - Issues
      - Each line is layer, use /'s
      - Don't leave build tools in images
      - Builder pattern before - at least 2 Dockerfles needed, scripts to copy files from one to the other
    - MSB to rescue - they allow multiple FROMS in Dockerfile so right there can COPY from one to the other
      - smaller prod images, only prod files copied
      - build image auto cleaned
  - Best practices
    - leverage build cache
    - when building images, checks to see if layer(s) exist(cache hit, cache miss)
    - If cache miss, cache no longer used for rest of build, so put instructions likely to change towards end of Dockerfile
    - So COPY and ADD doesn't cache miss, thinking no changes to layer but files changed, Docker does checksum on each file copied to it in the cached layer
    - --squash image not really a best practice
    - no-install-recommends - use wqith package manager like apt-get so only main dependencies are installed and not recommended/suggested ones (i.e. in Depends field)
    - Don't install MSI pachages - not efficient, larger images result
* The commands
  - Docker image build
  - FROM
  - RUN
  - COPY
  - EXPOSE
  - ENTRYPOINT
  - LABEL, ENV, ONBUILD, HEALTHCHECK, CMD, etc