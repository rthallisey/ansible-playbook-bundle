# The structure of an ansibleapp

### Directory layout
```
AnsibleApplication/
    Dockerfile
    ansibleapp/
        actions/
            provision.yaml
            deprovision.yaml
            bind.yaml
            unbind.yaml
```

These playbooks are called when their respective action is passed into the ansibleapp meta container. 

##### provision.yaml

This playbook should contain all tasks necessary to launch your application, including launching the containers and all configuration of services/routes/etc that are necessary for your application to work. If you used ansible-container to create your projects, the playbook generated by the shipit command should be sufficient.


##### deprovision.yaml

This playbook should contain all tasks necessary to tear down your application. In the majority of cases, this will be a single task that destroys the project, which will result in the deletion of all associated containers, services, and routes.

##### bind.yaml

TODO

##### unbind.yaml

TODO

### Example Dockerfile
```
FROM ansibleapp/ansibleapp-base

MAINTAINER Dylan Murray <dymurray@redhat.com>

ADD ansibleapp/actions /ansibleapp/actions
```

This Dockerfile is based off ansibleapp/ansibleapp-base which has an entrypoint script which will call the respective playbook out of /ansibleapp/actions/ as well as handling authentication.

Running an Ansibleapp
```
docker run -e "OPENSHIFT_TARGET=<oc_cluster_address>" -e "OPENSHIFT_USER=<oc_user>" -e "OPENSHIFT_PASS=<oc_pass>" <ansibleapp_name> $action
ex: docker run -e "OPENSHIFT_TARGET=cap.example.com:8443" -e "OPENSHIFT_USER=admin" -e "OPENSHIFT_PASS=admin" ansibleapp/etherpad-ansibleapp provision
```

# Using ansible-container to create an ansibleapp

[ansible-container](github.com/ansible/ansible-container) is a project that allows you to define a containerized application in yaml, and configure the individual containers using ansible. This section will assume that you have created an ansible-container project as described in the [ansible-container documentation](http://docs.ansible.com/ansible-container/). 

Once you have a working ansible-container project, creating an ansibleapp is trivial.

1. Push your images to your registry using [ansible-container push](http://docs.ansible.com/ansible-container/reference/push.html)
1. Generate a playbook and role for your application using [ansible-container shipit](http://docs.ansible.com/ansible-container/reference/shipit.html)
1. In the same directory as your ansible directory, create the ansibleapp/actions directory.
1. Copy the playbook generated by shipit (ansible/\<engine\>-shipit.yml) to ansibleapp/actions/provision.yaml
1. Create the ansibleapp/actions/deprovision.yaml file, and populate it with the content:

    ```yaml
    - hosts: localhost
      gather_facts: false
      connection: local
      tasks:
      - name: Delete <project-name> project
        command: oc delete project <project-name>
    ```
    deprovision.yaml may need to be further customized, depending on your application
1. Create a Dockerfile with the content:

    ```
    FROM ansibleapp/ansibleapp-base

    ADD ansible /usr/local/ansible
    ADD ansibleapp/actions /ansibleapp/actions
    ```
    
1. Your directory structure now should look something like this:

    ```
AnsibleApplication/
    Dockerfile
    ansible/
        meta.yml
        container.yml
        main.yml
        roles/
            AnsibleApplication-<engine>/
                defaults/main.yml
                tasks/main.yml
                meta/main.yml
    ansibleapp/
        actions/
            provision.yaml
            deprovision.yaml
    ```
1. Now just run `docker build . -t <container-name>`

You should now have a working ansibleapp. To provision/deprovision your application, simply run

```
docker run -e "OPENSHIFT_TARGET=<oc_cluster_address>" -e "OPENSHIFT_USER=<oc_user>" -e "OPENSHIFT_PASS=<oc_pass>" <container-name> provision
docker run -e "OPENSHIFT_TARGET=<oc_cluster_address>" -e "OPENSHIFT_USER=<oc_user>" -e "OPENSHIFT_PASS=<oc_pass>" <container-name> deprovision
```