Jenkins job builder
===================

Some templates for creating jenkins jobs via `puppet jobs builder
<http://ci.openstack.org/jenkins-job-builder/index.html>`_


Install the actual jenkins-job-builder command itself with::

    pip install jenkins-job-builder


``jenkins-jobs-builder`` expects a config file to provide the user credentials;
the jenkins servers don't actually use auth at this point but the file needs to
exist anyway and so we use jenkins_jobs.ini with dummy values.


To convert a template file into the xml format as expected by the jenkins api::

    jenkins-jobs --conf jenkins_jobs.ini test -o jenkouts .


To create/update a job on jenkins::

    jenkins-jobs --conf jenkins_jobs.ini update template.yaml


jenkins-job-builder creates a cache of jobs it has seen in order to prevent
needless api requests but the caching key only takes into consideration the
content of the template file and not what server the job was built on. This
means that when a new jenkins server is built any previosly run and cached jobs
will not be executed again. The simple fix is to simply remove the cache file::

    rm ~/.jenkins_jobs_cache.yml

Building a project
------------------

The jenkins templates are written to expect a `jenkins.sh` file in the root of
the workspace that will be used to execute the build commands. An example can
be seen in `gepetto
<https://github.com/adaptivelab/gepetto/blob/develop/jenkins.sh>`_ or `caiman
<https://github.com/adaptivelab/caiman/blob/develop/jenkins.sh>`_.

Configuring a Project
---------------------

To add new python jobs you can simply add a new project section to
`template.yaml` and simply update the `name` and `giturl` fields to
appropriate values. Adding jobs for other languages/platforms for where the
pep8 and pylint plugins are not applicable would require some investigation.


Puppetmaster
------------

Building a jenkins server consists of three parts, this jenkins job builder
for managing jobs, `gepetto <https://github.com/adaptivelab/gepetto>`_ for
spinning up the ci servers and `adaptivelab/puppet
<https://github.com/adaptivelab/puppet>`_ for provisioning the puppet servers.

To go from 0 to running ci server::

    pip install git+https://github.com/adaptivelab/gepetto@develop
    python -m gepetto.up

Gepetto will have created a new instance with puppet installed that will
perform some bootstrapping and attempt to connect to the puppetmaster. The
puppetmaster is no more however and so we need to create one. For now we can
create a puppetmaster on the same ci server::

    # somehow ascertain the address of the newly created gepetto server
    git clone https://github.com/adaptivelab/puppet
    scp -r puppet ubuntu@gepettoserver:~/
    ssh ubuntu@gepettoserver
    cd puppet

    # apply the puppetmaster config. this creates a puppetmaster that accepts
    # and signs all requests so it's important that random servers are not
    # allowed to connect
    sudo puppet apply --modulepath=modules -e "include puppetmaster"

    # if you're interested in the output you can stop the previously installed
    # puppet agent and run it in the foreground to see what happens. This will
    # set the current instance with the requirements to run as our ci server
    # Left to its own devices the agent should connect to the master but this
    # is untested and relies on correctly setup hostnames (gepetto's bootstrap
    # *did* correctly set the hosts at one point but the network architecture
    # has changed now) so manual running might be best for now
    sudo service puppet stop
    sudo puppet agent --verbose  --logdest console --no-daemonize --server=$(hostname)
    sudo service puppet start

The unscriptable part::

    sudo su jenkins
    ssh-keygen

Copy the contents of /var/lib/jenkins/.ssh/id_rsa.pub and add a new ssh key
for the admin+githubjenkins user on github

You can now see jenkins running on port 8080 in your browser but there are no
jobs. Update jenkins_jobs.ini with the new url, delete
`~/.jenkins_jobs_cache.yml` if it exists and run the commnads from above to
create some new jobs
