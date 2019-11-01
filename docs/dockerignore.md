# Dockerignore

Similar to the popular `.gitignore` file, `.dockerignore` uses regex expressions to tell docker what files/directories to not sync or touch.

If we had a Node application that has varying Node package dependencies, these dependencies should really be installed and maintained by our package manager (e.g. Yarn), and not copied/synced between local and container.

Using `.dockerignore`, we can tell docker to disregard the `node_modules` directory, making our container mappings and sync a lot more efficient and faster.

## Example

In the below example will configure docker to ignore:

- any and all files/folders in the `node_modules` directory
- any file with a extension of `.log`
- our environment file


```dockerignore
node_modules/
*.log
.env
```
