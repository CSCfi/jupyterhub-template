# GitHub OAuth

Deploys JupyterHub with GitHub OAuth as the authentication method.

## Creating GitHub OAuth application

Using the template requires an existing GitHub OAuth application. In the OAuth application, set:
- `homepage URL` as `https://<jupyterhub appliaction name>-<namespace>.rahtiapp.fi`
- `authorization callback URL` as `https://<jupyterhub application name>-<namespace>.rahtiapp.fi/hub/oauth_callback`

You need the following details from the OAuth application. Make sure you have them when using the template
- `client ID`
- `client secret`
- `authorization callback URL` 

More about creating GitHub OAuth applications can be found [here](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app).

## Using the template with oc CLI tool

Login to Rahti
```bash
$ oc login https://rahti.csc.fi:8443 --token=<token from Rahti>
```

Create a new project
```bash
$ oc new-project <your Rahti project name> --description="csc_project:<your CSC project name>"
```

After creatig the OAuth application, deploy JupyterHub using the template
```bash
$ oc new-app -f <path/to/template> -p PARAM1=Value1 -p PARAM2=Value2
```


### Parameters to be supplied

(Note: memory limits should only be changed after checking project quota to avoid errors)
| Parameter                       | Description                                                                                    | Default value |
| ------------------------------- | ---------------------------------------------------------------------------------------------- | ------------- |
| APPLICATION_NAME                | Name of the JupyterHub application                                                             | jupyterhub    |
| JUPYTERHUB_IMAGE                | Link to the JupyterHub image                                                                   | cscfi/jupyterhub-quickstart |
| NOTEBOOK_IMAGE                  | Link to the Notebook image                                                                     | quay.io/jupyteronopenshift/s2i-minimal-notebook-py36:2.5.1 |
| OAUTH_CLIENT_ID                 | Client ID of your GitHub OAuth application                                                     | (no default)  |
| OAUTH_CLIENT_SECRET             | Client secret of your GitHub OAuth application                                                 | (no default)  |
| OAUTH_CALLBACK_URL              | Callback URL of your GitHub OAuth application (in the form `https://<app name>-<namespace>.rahtiapp.fi/hub/oauth_callback`) | (no default) |
| JUPYTERHUB_ADMIN_USER           | GitHub username of default admin user on JupyterHub                                            | (no default)  |
| JUPYTERHUB_API_TOKEN            | API token for JupyterHub admin user                                                            | (no default, generated automatically if left empty) |
| JUPYTERHUB_ALLOWED_ORGANIZATION | Name of GitHub organization to whitelist                                                       | (no default)  |
| JUPYTERHUB_CULL_IDLE_TIMEOUT    | Time (in seconds) after which idle servers are culled (leave empty if no culling is desired)   | 3600          |
| COOKIE_SECRET                   | Encryption key used to encrypt JupyterHub browser cookies                                      | (no default, generated automatically if left empty) |
| DATABASE_PASSWORD               | Password for JupyterHub database                                                               | (no default, generated automatically if left empty) |
| JUPYTERHUB_MEMORY               | Amount of memory available to JupyterHub                                                       | 512Mi         |
| DATABASE_MEMORY                 | Amount of memory available to PostgreSQL                                                       | 512Mi         |
| NOTEBOOK_MEMORY                 | Amount of memory available to each notebook                                                    | 512Mi         |
| NOTEBOOK_VOLUME_SIZE            | Amount of storage available to each notebook                                                   | 1Gi           |
| DATABASE_VOLUME_SIZE            | Amount of storage available to database                                                        | 1Gi           |


## Configuring JupyterHub

The default configuration file of JupyterHub (`jupyterhub_config.py`) is stored in a ConfigMap called `<APPLICATION_NAME>-cfg`. To see changes made to the config file, you need to redeploy the JupyterHub deployment.

#### Adding new admins

New admins can be added through the JupyterHub web UI on the admin view of the control panel or by editing the `c.Authenticator.admin_users` option in the config file:
```python
c.Authenticator.admin_users = { os.environ['JUPYTERHUB_ADMIN_USER'], 'username-of-admin', 'another-admin' }
```

#### Adding new organizations to whitelist

If you want more than one organization to be whitelisted, edit the `c.Authenticator.allowed_organizations` option in the config file:
```python
if os.environ.get('JUPYTERHUB_ALLOWED_ORGANIZATION'):
    c.Authenticator.allowed_organizations = { os.environ['JUPYTERHUB_ALLOWED_ORGANIZATION'], 'name-of-organization', 'another-organization' }
```

#### User based whitelisting

If you want to use user based whitelisting, add the following line to the config file:
```python
c.Authenticator.allowed_users = {'list','of','users'}
```


## Adding Python packages to notebooks

The ConfigMap `<APPLICATION_NAME>-req` has a `requirements.txt` file which you can use to add Python packages into notebooks.

Note that new packages will only be available to users if their notebook servers are restarted. Servers can be stopped either through the admin view of JupyterHub's web UI (at `/hub/admin`) or by killing the notebook pods. They can be restarted by logging in again.
