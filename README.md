# Rāpoi-docs

The docs are organised in markdown files in `docs/`  The navigation pannel is configured in `mkdocs.yml` in the repository root.

If you'd like to see a local version before making a pull request or commit, see [Local mkdocs instructions](local_mkdocs.md).  This shouldn't be nessesary unless you're doing something complicated.  



## Admins

To actually deploy the docs all that's needed is a commit (either directly or accepting a pull request) to the master branch.  A github CI/CD pipeline will setup a temporary enviroment that pip installs the mkdocs dependancies and builds the site.  The config of the pipeline is in `.github/workflows/ci.yml`. If any additional mkdocs dependancies or plugins are required, add them there. 

After the docs have been deployed you may need to refresh your browser cache to see changes.