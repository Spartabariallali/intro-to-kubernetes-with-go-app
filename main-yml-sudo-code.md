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
- first we run the `minikube status` command and store it in the `minikube_status` variable
- the second `name` instructions set out that in there is no output from the minikube status command, or if it does not contain the word 'Running' in it, the task will run the command `minikube start` and for the command to complete

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

## 3. Building the container images in minikube with ansible
- the first task checks if there is an image wiht the `image_name` variable defined earlier.
- Because we are running the playbook on `localhost`, but minikube has its own docker environment, we use ansible's shell module to redirect the minikube environment.
- In the `shell` command we use the vertical pipe `|` to indicate to the YAML parser it should store the following lines as multi-line scalar.
- we store the output in `image_hash` from the `docker images` command, we can build the docker image - but only if it not already built.
- the `when` condition says "if theres no stdout returned from the `docker images` command assume the image doest exist yet."
- Because this playbook is in a director adjacent to the `hello-go` example (which contains the `Dockerfile`)
- the context passed to the `docker build` command is `../hello-go` this directs Docker to look for a dockerfile inside the hello-go directory adjacent to this playbok's `hello-go-automation` directory.

```
tasks:
  # Build the hello-go Docker image inside Minikube's environment.
  - name: Get existing image hash.
    shell: |
      eval $(minikube docker-env)
      docker images -q {{ image_name }}
    register: image_hash
    changed_when: false

  - name: Build image if it's not already built.
    shell: |
      eval $(minikube docker-env)
      docker build -t {{ image_name }} ../hello-go
    when: not image_hash.stdout
```

## 4. Installing OpenShift to create Kubernetes resource
- `brew install openshift-cli`
- Ansible makes it easy to manage resources with its k8s module. the module uses openshift python client to interact with kubernetes' API.
- Once its installed you can pass a full Kubernetes resource defintion to the k8s module.

```
# Create Kubernetes resources to run Hello Go.
- name: Create a Deployment for Hello Go.
  k8s:
    state: present
    definition:
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: hello-go
        namespace: default
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: hello-go
        template:
          metadata:
            labels:
              app: hello-go
          spec:
            containers:
            - name: hello-go
              image: "{{ image_name }}"
              imagePullPolicy: IfNotPresent
              ports:
              - containerPort: 8180
```


## 5. Create service that exposes via cluster LoadBalancer

```
- name: Create a Service for Hello Go.
  k8s:
    state: present
    definition:
      apiVersion: v1
      kind: Service
      metadata:
        name: hello-go
        namespace: default
      spec:
        type: LoadBalancer
        ports:
        - port: 8180
          targetPort: 8180
        selector:
          app: hello-go
  ```

## 6. Exposing the service to the host using minikube Service

```
post_tasks:
  - name: Expose Hello Go on the host via Minikube.
    command: minikube service hello-go --url=true
    changed_when: false
    register: minikube_service

  - debug:
      msg: "Hello Go URL: {{ minikube_service['stdout_lines'][0] }}"
```
