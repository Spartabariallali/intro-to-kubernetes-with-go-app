## Main.yml sudo code explanation

## 1. Creating variables in the playbook
- The `vars` section defines the variables that will be used in the playbook.
- The first variable `ansible_playbook_python` is set this way to ensure all tasks run on the local machine inherit the same Python environment that used by the ansible-playbook command.
- The second varibale `image_name` is used to name the container image when we build it and deploy it in Minikube.

```
---
- hosts: localhost
  gather_facts: false

  vars:
    ansible_python_interpreter: '{{ ansible_playbook_python }}'
    image_name: hello-go
```

## 2. Pre_task
- After we set the varibales, we ensure minikube is running in a `pre_task` section of the playbook.
- `pre_tasks` will always run before tasks


```
pre_tasks:
  - name: Check Minikube's status.
    command: minikube status
    register: minikube_status
    changed_when: false
    ignore_errors: true

  - name: Start Minikube if it's not running.
    command: minikube start
    when: "not minikube_status.stdout or 'Running' not in minikube_status.stdout"
```
