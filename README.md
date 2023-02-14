# Mstaff micro apps POC

This POC has been made to demonstrate how we can split our application into multiple micro applications (frontend for now).

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

## How to install

### Prerequisites

Have Docker.

### Cloning

Clone the main project.

### Run containers

In ``apps`` directory, go in each app you want to start and run ``docker-compose up -d``.

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

## How it works

### About structure

All apps are in ``apps`` directory.\
All shared pieces of code (submodules) are in ``shared`` directory. This is for demonstration purposes and not realy required. You can either modify those submodules from those directories or directly from within a project.\
*More about submodules below.*

All project are for now using their respective technologies dev servers to be served.
- So ``webpack-dev-server`` for Vue apps : https://webpack.js.org/configuration/dev-server/.
- And ``Symfony Local Web Server`` for Symfony apps : https://symfony.com/doc/current/setup/symfony_server.html.