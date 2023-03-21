---
layout: post
title:  Developing command line tool for build & deployment
date:   2020-04-20
description: 
tags:
- cli
- python
- jenkins
- gitops
- docker
permalink: cli-tool-for-build-deploy
---

CI/CD in software engineering is a process to apply developer's work to project. It includes maintaining code, compiling process, testings(unit, e2e), and all other things for software updates. This phrase has been used more than 20 years, and various tools for integration/deployment has been created during 2 decades.

![Screenshot](/assets/post_img/cli-tool-for-build-deploy/jenkins-docker-logo.png)

Jenkins is open-source automation server for continuous integration/delivery of project. It is one of most popular tools for CI/CD, and I've also worked with this for several years. 

Containerizing has been commonly used in software development(powered by docker, k8s) from several years ago, and jenkins also applied features for treating containerized project.

Current service I'm working on, is managed on Kubernetes(k8s) cluster, with various docker images.


## Motivation
![Screenshot](/assets/post_img/cli-tool-for-build-deploy/jenkins-home.png)

By jenkins main page, you can monitor current build pipelines, and create new one if you needed. But if project becomes bigger, jobs and pipelines will be pile up, and finding your target job from GUI dashboard will become inconvenient. Well, actually this is pretty simple process comparing with old-fashion logics, but anyway I wanted to make it more simple.

Thankfully, jenkins offers APIs to control functions and monitor job/pipeline status through HTTP request/response. So I've planned to make build-deploy process through command line interface(CLI) so not need to access dashboard for software updates.

I've implemented and using it well as in-house private project. It saved lot of time accessing dashboard, going inside target pipeline, and monitoring the build status.


## Life is short, you need Python
Cannot say Python is a best programming language, but at least it is best option to me, for developing quick scratch. It helps developer to create logics with short lines comparing with other languages, and that's why people said `Life is short, you need Python`. Moreover, jenkins are offering wrapped-up API for python, so you don't need to use http call directly.

Also there are great packages for CLI developments, so it helped me to make this fast and neat. Let's see the basic settings for making package. There are several packages for CLI, so I've used [click](https://click.palletsprojects.com)(I've used 7.x) for this plan.

Let's name this project as `mycli`. I've setup 'click', and [python-jenkins](https://python-jenkins.readthedocs.io/en/latest/) package to access jenkins.

Setup virtual-env, add packages on `requirements.txt`, and create `setup.py` to define package.
```python
with codecs.open(os.path.join(here, 'requirements.txt')) as f:
    install_requirements = [
        line.split('#')[0].strip() for line in f.readlines() if not line.startswith('#')
    ]

setup(
    name='mycli',
    version='0.1.0',
    description='bla bla bla',
    install_requires=install_requirements,
    py_modules=['mycli'],
    entry_points='''
        [console_scripts]
        mycli=mycli:cli
    ''',
)
```

Now make `mycli.py`, and let's define command to call user information. You need to get API token from user config page.
```python
import click
import jenkins


@click.group()
def cli():
    pass

# put command name as parameter in 'cli.command' if you want to use other name for command
@cli.command() 
def user():
    """
    Get user information
    """
    host = 'https://jenkins-url'
    try:
        server = jenkins.Jenkins(
            host,
            username='user-email',
            password='user-api-token'
        )
        user_info = server.get_whoami()
        click.echo('Jenkins user:' + str(user_info))
    except Exception as e:
        raise Exception(e)
```

Basic configuration are prepared. Install packages and try `user` command to check this works well.
```
$ pip install -e .
...
$ mycli user
...
Jenkins user: ....
```


## Design process for build and git-ops deployment
GitOps is a way of implementing deployments for cloud native applications, by controlling containers with separated git repository.

> The core idea of GitOps is having a Git repository that always contains declarative descriptions of the infrastructure currently desired in the production environment and an automated process to make the production environment match the described state in the repository. If you want to deploy a new application or update an existing one, you only need to update the repository - the automated process handles everything else...

So I've designed flow as below:

![Screenshot](/assets/post_img/cli-tool-for-build-deploy/jenkins-gitops.png)

and planned to create command for `build`(center part of image), and `deploy`(right side of image).
```python
@cli.command('build')
def build_job():
    """
    Build jenkins job
    """
    host = 'https://jenkins-url'
    try:
        server = jenkins.Jenkins(
            host,
            username='user-email',
            password='usser-api-token',
            timeout=30
        )
        queue_number = server.build_job('project-name')
        from time import sleep; sleep(10)
        build_info = server.get_build_info('project-name', next_build_number)
        job_info = server.get_job_info(queue_number)
    except Exception as e:
        raise Exception(e)
```

By definition of your project, either you can need build info or job info after you triggered the build. Because build takes few seconds to boot-up after started, you need to define delay(usually 10 seconds are OK) before getting build informations.


## Control git-ops repository
After build has been completed, you need to update container information through config files managed by git. So `deploy` command will be:
> 1. 'git pull' for get newest configuration
> 2. Update configuration file
> 3. 'git push' to push changed configuration

As you expected, there are also packages for git operation [GitPython](https://gitpython.readthedocs.io/en/stable/). Install and try:

```python
from git import Repo

@cli.command('deploy')
def deploy_container():
    """
    Deploy container through git
    """
    repo_url = 'https://git-ops-repo.url'
    repo = Repo(repo)
    repo.git.checkout('master')
    repo.git.pull()

    # fix configuration file
    # ...

    repo.index.add(['list-of-changed-file'])
    repo.index.commit('commit-message')
    origin = repo.remote(name='origin')
    origin.push()
```

This is how to control repository by commented order. It will pull the repository in master branch, (fix container image info in configuration file to new one), and push it to apply cluster. You need to put your logics to update newest build status in configuration files(usually defined as `.yml` or `.json`) at commented part.


## Conclusion
Now, let's check the tool:
```
$ mycli --help                                                        ⏎
Usage: mycli [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  build   Build jenkins job
  deploy  Deploy container through git
  user    Get user information
```

As you can expected, this is concise version of real tool I'm using on. But because it includes major features, you can refer it to make your own tool which can help your development life just as it did to me.


## Reference
* <https://www.jenkins.io/>
* <https://www.gitops.tech/>
* <https://click.palletsprojects.com>

