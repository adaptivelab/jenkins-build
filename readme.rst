Jenkins job builder
===================

Some templates for creating jenkins jobs via `puppet jobs builder <http://ci.openstack.org/jenkins-job-builder/index.html>`_


Install the actual jenkins-job-builder command itself with:

    pip install jenkins-job-builder


``jenkins-jobs-builder`` expects a config file to provide the user credentials;
the jenkins servers don't actually use auth at this point but the file needs to
exist anyway and so we use jenkins_jobs.ini with dummy values.


To convert a template file into the xml format as expected by the jenkins api:

    jenkins-jobs --conf jenkins_jobs.ini test -o jenkouts .


To create/update a job on jenkins:

    jenkins-jobs --conf jenkins_jobs.ini update template.yaml


jenkins-job-builder creates a cache of jobs it has seen in order to prevent
needless api requests but the caching key only takes into consideration the
content of the template file and not what server the job was built on. This
means that when a new jenkins server is built any previosly run and cached jobs
will not be executed again. The simple fix is to simply remove the cache file:

    rm ~/.jenkins_jobs_cache.yml
