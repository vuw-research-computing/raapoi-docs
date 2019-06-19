# raapoi-docs
Documentation for the Rāpoi cluster.

Built using the sphinx-based Read the Docs theme and deployed via Mkdocs. mkdocs.yml configures basic site structure, menu structure and Table of Contents depth, and can also customise other variables. Extra.css defines non-standard styling. 

Currently, heading levels 1 and 2 (# and ##) creating clickable menu items in the side nav bar.

##Workflow for updating docs:

1. Make sure you're on the ```master``` branch on raapoi-docs repository.

1. Naviate to the ```docs``` folder and open and edit (or create) the appropriate ```.md``` file, (e.g. ```examples.md```).

1. Save this once changes are made, then ```add```, ```commit```, and ```push``` this to ```origin master```.

1. Now deploy this with MKdocs commands:

```mkdocs build --clean``` 
(This builds the markdown files into a ```/site``` folder, where each ```.md``` files is a folder with it's own ```index.html```). ```clean``` overwrites and cleans existing ```/site``` directory. 
1. to see changes locally, use ```mkdocs serve```

1. When ready to publish, use:
```mkdocs gh-deploy``` (optional ```--clean``` can be appended to overwrite past )

This should return something like: 
```
plummema@ITS-7MTSF2S MINGW64 /h/GIT_HUB/raapoi-docs (master)
$ mkdocs gh-deploy
INFO    -  Cleaning site directory
INFO    -  Building documentation to directory: H:\GIT_HUB\raapoi-docs\site
INFO    -  Copying 'H:\GIT_HUB\raapoi-docs\site' to 'gh-pages' branch and pushing to GitHub.
INFO    -  Your documentation should shortly be available at: https://vuw-research-computing.github.io/raapoi-docs/
```
That's it. If editing multiple files at once, make local changes on all the required files first then these steps only need be worked through once.

