ansible-mesos-playbook
======================

An ansible playbook for launching a mesos cluster with native docker and mesos executors, along with [Marathon](https://github.com/mesosphere/marathon)) and HAProxy support. Run this on Ubuntu 14.04 LTS (preferred) or Centos/RHEL 6. [Read the blog post](http://blog.michaelhamrah.com/2014/06/setting-up-a-multi-node-mesos-cluster-running-docker-haproxy-and-marathon-with-ansible/) for a descriptive overview.

### Getting Started

* Install [librarian-ansible](https://github.com/bcoe/librarian-ansible) via ```gem install librarian-ansible```
* Run ```librarian-ansible install```
* Spin up a bunch of Ubuntu 14.04 servers, say 5, on your favorite cloud provider.
* ```cp hosts.sample hosts``` and update the ```mesos_masters``` and ```mesos_slaves``` groups.
* ```cp ansible.cfg.sample ansible.cfg``` to ensure librarian_roles is in the ansible path (```ansible.cfg``` is git-ignored).
* Run ```ansible-playbook -i hosts playbook.yml```.

### The Setup

* zookeeper, haproxy, mesos-master and marathon with ha mode run on nodes in the mesos_master group. The zoo_id host variable is used to configure zookeeper.
* mesos-slave runs in the mesos_slaves group and are passed the list of mesos_masters for coordination.
* Deimos is installed on the slaves as the external container executor for Mesos.
* A cron job on each master is set up to query the marathon api and configure HAProxy.
* HAProxy routes a frontend (listening on port 80) to backends based on marathon tasks.
* You probably want to tweak the HAProxy configuration script (in /opt/marathon/bin) for your needs. With the current setup you can have a wildcard dns prefix route to a backend matching the marathon name: i.e. www.example.com would do a least-connection proxy to the www task.

### Notes

Currently this installs Mesos 0.20 with Marathon 0.7.0-RC3. Mesos 0.20 only supports host networking, so make sure your containers pick up and use the assigned Marathon ports. 

### Launching a Container

POST to /v2/apps:

```
{
    "id": "www", 
    "container": {
        "docker": {
            "image": "mhamrah/mesos-sample"
        },
        "type": "DOCKER",
        "volumes": []
    },
    "args": ["hello"],
    "cpus": 1,
    "mem": 512,
    "instances": 1,
  "ports": [0]
}
```
