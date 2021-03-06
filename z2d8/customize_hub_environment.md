# Customizing the JupyterHub environment for Data 8

The Data 8 course uses a collection of Python modules and open-source
technology for both course infrastructure as well as teaching. We've
provided all of these materials as a Docker image that you can connect
to your JupyterHub so that all users will have the environment needed
for the class. This page describes how to set up your JupyterHub
environment so that it serves the same environment that the Data 8
students have.

## Deploying a change to your JupyterHub configuration

All modifications to the base JupyterHub deployment are made with
changes to your `config.yaml` file. When we first [created our JupyterHub](setup_k8s.md),
we created this file with a single value inside that contained our secret token.
This section shows how you can customize your JupyterHub for Data 8 by modifying
your `config.yaml` file.

**Once you've made a change to `config.yaml`** you can deploy it with the following
steps:

1. **Check that your `config.yaml` has no duplicate headers.** For example, you may have
   copy/pasted two code snippets that both had
   
   ```
   singleuser:
   ```
   
   As the top header. Make sure that these are merged under a *single header*.
2. **Deploy your change to the JupyterHub.** Use the following command:

    ```
    helm upgrade {{ YOUR-HUB-NAMESPACE }} jupyterhub/jupyterhub --version=v0.6 -f config.yaml
    ```


    This runs a "Helm Upgrade", which tells Kubernetes to update its deployment to match
    the values that you've placed in `config.yaml`. The value in `{{ YOUR-HUB-NAMESPACE }}` should
    be whatever you chose when [creating the JupyterHub](setup_k8s.md).
    
    Most hardware modifications to your
    Kubernetes deployment will be done in this way.

**If you get a "time out waiting for the condition" error** then it took too long to
pull the Docker image onto the JupyterHub. You can generally resolve this by including
a `--timeout=9999999` flag to your `helm upgrade` command. This will prevent Helm
from stopping too early.

The following sections cover some common things that need to be done to prepare your
deployment for Data 8.

## Using the Data 8 Docker image

First, we'll tell the JupyterHub to connect user sessions with the
Docker image used by Data 8. This is done by adding the following to your
`config.yaml` file:

```
singleuser:
  image:
    name: berkeleydsep/datahub-user
    tag: 21be6ff
```

To deploy the change, save the file, then run a helm upgrade:

```
helm upgrade data8 jupyterhub/jupyterhub --version=v0.6 -f config.yaml
```

Note that this will take a while if you're using the data8 image, perhaps
upwards of 10 minutes, as it pulls the image into your Kubernetes deployment.


## Define the resources available to your users

In this section we'll define what kind of resources your students have. For example,
how much RAM / CPU / etc. See the [Data 8 user resource configuration](https://github.com/berkeley-dsep-infra/datahub/blob/staging/datahub/config.yaml#L140)
for example.

### Memory (RAM)
The following snippet will give each user 1 gig of ram, and 2 gigs of storage,
which is the amount given to Data 8 students at Berkeley.

```
singleuser:
  memory:
    guarantee: 1G
    limit: 1G
  storage:
    capacity: 2Gi
```

### Disk Storage

The following snippet gives students 2 gigs of persistent storage:

```
singleuser:
  storage:
    capacity: 2Gi
```

## Authorization for your hub

Authorization allows you to control who has access to your JupyterHub, as well
as keep track of who is accessing the hub.

There are many options for
authorization with JupyterHub. The Data 8 deployment uses the Berkeley
CalNet authentication system, which is part of the Google "G Suite".
Your authentication approach depends on how your university authenticates.
You can also choose
authentication with popular services like GitHub.

See the [Zero to JupyterHub Authentication guide](https://zero-to-jupyterhub.readthedocs.io/en/latest/authentication.html) for information about various
authentication options and how to enable them.

For this guide, we'll show you how to authenticate using GitHub usernames,
as this is a free service that is more widely-accessible than the particular
authentication system that Berkeley uses.

### Authenticating with GitHub
To authenticate with GitHub, take the following the steps outlined
in the [Zero to JupyterHub Authentication Guide](https://zero-to-jupyterhub.readthedocs.io/en/latest/authentication.html#github).

Note: The DSEP authorization configuration [can be found here](https://github.com/berkeley-dsep-infra/datahub/blob/staging/datahub/config.yaml#L65).

### Adding admin users

JupyterHub has an **administrator** page that can be used to see all of the
active sessions currently on the hub, as well as perform some simple actions
to help debug and fix student problems. To add a list of admin usernames,
add the following to the `auth` section of your `config.yaml` file:

```
auth:
  admin:
    users:
      - <LIST>
      - <OF>
      - <ADMIN>
      - <USERNAMES>
```

## The final `config.yaml` file

If you've followed all of the instructions on this page
(including authenticating with GitHub), your `config.yaml` file should now
look something like this:

```
proxy:
  secretToken: "{{ output of openssl rand -hex 32 }}"

singleuser:  # This defines the user environment
  image:
    name: berkeleydsep/datahub-user
    tag: da80cb1
  memory:
    guarantee: 2G
    limit: 2G
  storage:
    capacity: 2Gi
    
auth:
  type: github
  github:
    clientId: "{{ YOUR-CLIENT-ID }}"
    clientSecret: "{{ YOUR-CLIENT-SECRET }}"
    callbackUrl: "http://{{ YOUR-HUB-IP-ADDRESS }}/hub/oauth_callback"
  admin:
    users:
        - <LIST>
        - <OF>
        - <ADMIN>
        - <USERNAMES>
```

## Deploy your changes

Once you've configured `config.yaml` properly, you can upgrade your
JupyterHub with a Helm upgrade:

`helm upgrade data8 jupyterhub/jupyterhub --version=v0.6 -f config.yaml`

## Confirm that your environment works

To confirm that you're running the correct environment needed for Data 8,
take the following steps:

1. Go to your JupyterHub's public IP address. You can find this address with:

   ```
   kubectl --namespace=data8 get svc proxy-public
   ```

2. Log in with your username/password (if you haven't logged in yet, use whatever
   username/password you'd like)
3. Click "Start My Server", then create a new Jupyter Notebook.
4. Run the following Python code

   ```python
   import datascience
   print(datascience.__version__)
   ```

You should see the version for the `datascience` package printed below the cell.
If this worked, then congratulations! Your JupyterHub is ready to go.

## Next: connect with course materials

Now that we've customized our environment to work with Data 8, it's time
to connect with the course materials such as the textbook, labs, and homeworks.
To do so, [go to the next section](connect_class_materials.md).