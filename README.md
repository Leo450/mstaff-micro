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

TODO