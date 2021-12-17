---
title: A GitHub action to turn Jupyter notebooks into Minimal Mistakes Jekyll blog posts
tags: ["Minimal Mistakes", "Jekyll", "Jupyter", "GitHub Actions"]
category: "Blogging platform"
---

# Starting point
As mentioned in [Turning Jupyter notebooks into Jekyll blog posts]({% post_url 2021-12-17-jupyter-blogs %}), I was deploying my Minimal Mistakes Jekyll site to the
`gh_pages` branch using a simple workflow. I now wanted to add an action which would take
all notebooks stored in a specific folder and turn them into markdown posts.
```
name: Deployment

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  build-site:
    runs-on: ubuntu-latest
    steps:
      # Check out current repo
      - uses: actions/checkout@v2

      # TODO HERE:
      # Action which turns notebooks into Markdown posts

      # Build and deploy the site
      - name: Build and deploy
        uses: EdricChan03/action-build-deploy-ghpages@v2.5.0
        env:
          JEKYLL_ENV: 'production'
        with:
          github_token: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
```

I had also written a simple script consisting of a few lines of bash and perl and a call to
`jupyter nbconvert` to convert notebooks to Markdown posts, taking care of moving the file
and the images and fixing image links. Finally, I found that creating a new layout type
`notebook` was the easiest way to include badges to view the notebook on GitHub or Google
Colab.

I decided to bundle the two files which are required on top of the typical Jekyll files into
the action, such that it could be used with zero installation.

One seemingly small detail that doesn't seem to be too clearly documented is how data can be
accessed in the different stages of the action. Here is what I think to have understood:
* When setting up the docker container (when running the `Dockerfile`), the files in the action
  repository are available. We can therefore copy any files into the container at that point.
* The container will start in the ${GITHUB_WORKSPACE} which is mounted at `/github/workspace`.
  The files in the action repository are not available anymore, unless we have copied them
  previously.
* ${GITHUB_WORKSPACE} is also where, for example, `actions/checkout` is checking out the
  repository in. The workspace remains available throughout the steps of a job, which means that
  it can be used to store files which will be used by later actions.

# Setting up a docker container
The action has relatively few requirements: We need an environment to run bash and perl and
we need to be able to use `jupyter nbconvert`. I decided to use `jupyter/base-notebook`, which
is a basic installation of Jupyter on Ubuntu. It is pretty light-weight but has all
the features we will need pre-installed.

The `Dockerfile` is relatively easy, we only need to specify our starting point (the current
`base-notebook` release), add the `notebook` layout and the action script, and set
the action script as the entry point.

```docker
# Container image that runs our code.
# base-notebook is a ubuntu image with a light-weight Jupyter installation
FROM jupyter/base-notebook:2021-12-16
USER root

# The action needs the special notebook layout and the action script
# We'll just copy the entire folder there
COPY . /action

# Run the action script when starting the docker container
ENTRYPOINT ["/action/convert_notebooks.sh"]
```

A few notes:
* `jupyter/base-notebook` is released every day. For reproducibility, I picked a recent
  version. It's unlikely I'll have to change it any time soon.
* By default, the `jupyter/base-notebook` has no root privileges. GitHub actions do, however,
  normally run using the `root` user. The non-privileged default user does not have access to
  the ${GITHUB_WORKSPACE} folder which is the default work directory for GitHub actions. To
  play well with other actions, I decided to use the `root` user rather than trying to work in
  a different folder.
* When building the Docker container, we have full access to the files of the action. Since we
  need some of them later, I decided to simply copy the full folder to the `/action` location
  in the container.
* The entry point is the script which does the entire conversion. The container will be run from
  ${GITHUB_WORKSPACE}.

# Action metadata file
The action metadata file defines the name and description of the action, as well as the
run environment, the inputs, and the outputs. For our action, it is extremely easy since
we need no inputs and no outputs. All we need to define is the name and the description,
and that we are using docker to run it.

The minimal metadata file looks like this:
```yml
name: 'Jupyter Minimal Mistakes'
description: 'Convert Jupyter notebooks to Markdown blog posts ready for use with Jekyll and the Minimal Mistakes theme'
runs:
  using: 'docker'
  image: 'Dockerfile'
```

# Action script
The action script itself is a simple bash script which does some sanity checks, and then
loops over the notebooks found in the `_notebooks` folder and creates post for each of
them. It also creates the `notebook` layout in the `_layouts` folder if it doesn't exist.
The full action script is [here](https://github.com/ptmerz/jupyter-minimal-mistakes/blob/main/convert_notebooks.sh).

# Tests
I added some basic functionality and regression tests. The test checks out the current
repository and transforms a simple Jupyter notebook into Markdown. The resulting Markdown
file is checked against a reference file, and the existence of the `notebook` layout file
and the image file is verified. This is unlikely to catch all errors, but does at least
perform some basic functionality check.

# Notes
To run a base image locally and play around a bit, this command is very helpful:
```bash
docker run -it --entrypoint /bin/bash jupyter/base-notebook
```
This will start a docker container with the chosen base image (in the example above
`jupter/base-notebook`) with a bash session as entry point. This is very handy to run some
commands interactively and make sure that everything works as intended.

# References
[GitHub Docs: Creating a Docker container action](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action)
