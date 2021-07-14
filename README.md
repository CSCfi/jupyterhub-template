# jupyterhub-template

The templates are used to deploy JupyterHub on OpenShift.

JupyterHub brings the power of notebooks to groups of users. It gives users access to computational environments and resources without burdening the users with installation and maintenance tasks. Users can get their work done in their own workspaces on shared resources which can be managed efficiently by system administrators. You can read more about JupyterHub [here](https://jupyterhub.readthedocs.io/en/stable/).

There are two templates for deploying JupyterHub on OpenShift depending on the authentication method used:
- [GitHub OAuth](github-oauth.md)
- [Native authenticator](native-authenticator.md)

Both templates and the JupyterHub [image](https://github.com/CSCfi/jupyterhub-quickstart) used in them are based on https://github.com/jupyter-on-openshift/jupyterhub-quickstart.
