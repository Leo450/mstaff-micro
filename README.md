# Mstaff micro apps POC

This POC has been made to demonstrate how we can split our application into multiple micro applications (frontends for now).

The goal is to have multiple frontends for different part in a same application. The navigation between them must be seamless, the user must feel like he is still using the same app even if he is redirected from one to another.

This POC handle those problematics :

- **The apps can be made with different technologies**
  For now we have two `VueJS` apps and two `Symfony` apps

- **They need to share data between each other**
  This POC is testing sharing data through domain restricted `Cookies`

- **They must be able to share code**
  We are using git `submodules` to write code only once and clone it through projects.

## Why submodules over monorepo ?

Simple, submodules are an answer to only the code sharing problematic.\
Monorepos bring answers to more problematics as well but also come with other difficulties that we didn't want to address, especially about CI subjects.

We want each application to be standalone and not to have any kind of depency between each other.
To achieve that, we prefer Git submodules flexibility :

- You can simply choose which submodule you want to clone or not in each project
- Each submodule can be cloned wherever you want in each project
- Each submodule is separately version controlled, so you have way more control over which project is using which version of each submodule.\
  *You can for instance modify a submodule directly from within a project and push onto a different branch without impacting other projects until you explicitely tell other projects to use this branch.\
  Even if you merge this branch into the main branch, you have to manually update submodules in other projects to tell them to point to the last commit of the main branch.*

## How to install locally

### Prerequisites

Have Docker.

### Recap

**[Clone](#cloning)**
- ``git clone [REPOSITORY_URL] --recurse-submodules``

**[Fix detached HEAD](#detached-head)**
- ``cd`` your project
- ``git submodule foreach --recursive 'git checkout main'``

**[Run](#run-containers)**
- ``docker network create mstaff-micro.network``
- ```
  docker-compose -f apps/web-server/docker-compose.yml up -d &&\
  docker-compose -f apps/front-symfo-1/docker-compose.yml up -d &&\
  docker-compose -f apps/front-symfo-2/docker-compose.yml up -d &&\
  docker-compose -f apps/front-vue-1/docker-compose.yml up -d &&\
  docker-compose -f apps/front-vue-2/docker-compose.yml up -d
  ```

### Clone

Clone the main project by running ``git clone [REPOSITORY_URL] --recurse-submodules``.

Do not forget the ``--recurse-submodules`` option, it will automatically initialize and update each submodule in the repository, including nested submodules because apps submodules in the main repository have submodules themselves.

If you forgot to add the ``--recurse-submodules`` option, you can manually go into each app and either run ``git submodule init`` and ``git submodule update``, or just ``git submodule update --init`` which is a shortcut for the two previous commands. To also initialize, fetch and checkout any nested submodules, you can use the foolproof ``git submodule update --init --recursive``.

### Fix detached HEAD

The only issue here is that even if all your submodules have been initialized and updated, the pointers in your superprojects are just pointing to single commit hashes and not to branches, resulting in each submodule HEAD being detached from their ``main`` branch.\

We need to manually explicitly checkout the ``main`` branch of each submodule by running ``git submodule foreach --recursive 'git checkout main'``.

### Run

First, create docker network by running ``docker network create mstaff-micro.network``.

Then run this command to start all containers.
```
docker-compose -f apps/web-server/docker-compose.yml up -d &&\
docker-compose -f apps/front-symfo-1/docker-compose.yml up -d &&\
docker-compose -f apps/front-symfo-2/docker-compose.yml up -d &&\
docker-compose -f apps/front-vue-1/docker-compose.yml up -d &&\
docker-compose -f apps/front-vue-2/docker-compose.yml up -d
```

And this command to stop and remove them all.
```
docker-compose -f apps/web-server/docker-compose.yml down &&\
docker-compose -f apps/front-symfo-1/docker-compose.yml down &&\
docker-compose -f apps/front-symfo-2/docker-compose.yml down &&\
docker-compose -f apps/front-vue-1/docker-compose.yml down &&\
docker-compose -f apps/front-vue-2/docker-compose.yml down
```

Or go into each app you want to start and run ``docker-compose up -d`` to start its container and ``docker-compose down`` to stop it.

### Access apps

Once started, all apps are served through the ``web-server`` app under their own respective urls :
- Symfony 1 : ``http://localhost:20000/front-symfo-1``
- Symfony 2 : ``http://localhost:20000/front-symfo-2``
- Vue 1 : ``http://localhost:20000/front-vue-1``
- Vue 2 : ``http://localhost:20000/front-vue-2``

They are also directly served on some ports for dev purposes :
- Symfony 1 : ``http://localhost:21000``
- Symfony 2 : ``http://localhost:21001``
- Vue 1 : ``http://localhost:22000``
- Vue 2 : ``http://localhost:22001``

## How are my apps served locally

### About structure

All apps are in ``apps`` directory.\
All shared pieces of code (submodules) are in ``shared`` directory. This is for demonstration purposes and not realy required. You can either modify those submodules from those directories or directly from within a project.

### Apps servers

All project are for now using their respective technologies dev servers to be served.
- So ``webpack-dev-server`` for Vue apps : https://webpack.js.org/configuration/dev-server/.
- And ``Symfony Local Web Server`` for Symfony apps : https://symfony.com/doc/current/setup/symfony_server.html.

### Main Web Server

The ``web-server`` port ``20000`` is exposed and he is serving apps under sub urls via url rewriting. You can find Nginx configuration and those rules in ``apps/web-server/nginx.conf``.

Apps containers share a docker network named ``mstaff-micro.network``.\
This allows the ``web-server`` app, which is just a Nginx container, to point directly to other apps containers.

## What does it demonstrate

### Code sharing via submodules

- **Both ``Symfony`` apps are sharing ``Entities`` and ``Repositories``**
  It will be much simpler to ensure that all database structure modifications are reflected onto all our Symfony apps, if they are all plugged onto a same DB for instance.

- **Both ``VueJS`` apps are sharing ``UI Components``**
  It will be much simpler to create a standardised Design System and ensure all the apps look the same.

### Data sharing via Cookies and redirection

All apps must share data between each other.

The first use case that comes to mind is *authentication data*, like tokens or user data. Vue apps are considered as *authenticated* apps. You should be redirected to a Symfony app if you are not logged in.

- All apps all have buttons to create/remove a dummy ``token`` cookie.
- Vue apps have a ``beforeEach`` router middleware to check if the ``token`` cookie is set. If it's not, it is redirecting to Symfony app 1.

## More about using submodules

The simplest way to use submodules in my opinion, and the main thing to understand, is that when you ``cd`` into a submodule directory from within a superproject, you are basically working on this submodule repository like if it was just a simple repository. You can make changes, commit, push and pull as you are used to.

The only thing to remember is if you pushed changes to a submodule, the remote repository of the superproject is still pointing to the older commit of the submodule.\
If you go back into your superproject and run a ``git status`` it will show you that you have modifications on the submodules directories with the annotation ``(New commits)``.\
You need to also commit and push those modifications, which are just new submodules commit pointers, to you superproject repository. If you do so and go to Github, you will see on your submodules directories that the hash of the commit they point to just changed.\
![alt text](readme-img.png "submodule commit")

### In depth

https://git-scm.com/book/en/v2/Git-Tools-Submodules