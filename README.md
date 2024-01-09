# Podman Play - Deploy Any App

Ansible Role to deploy apps in root-less containers from a Kubernetes Pod YAML definition. The application pod runs as a systemd service, in your own user namespace.

**🔑 Key Features**
- **🚀 Deploy Any Application**: Easily deploy any application using a Kubernetes YAML pod definition.
- **🛡️ Root-less deployment**: Ensure secure containerization by running custom applications in a root-less mode within a user namespace. Management of the container is handled through a systemd unit.
- **🔄 Idempotent deployment**: Role embraces idempotent deployment, ensuring that the state of your deployment always matches your desired inventory.
- **🧩 Flexible Configuration**: Easily customize deployment configuration to match your specific requirements.

Check out series of blog posts to discover how simple it is to use this role for deploying popular applications:
 - Deploy [Dashy](https://dashy.to/), [Nextcloud](https://nextcloud.com/), [Hashi Vault](https://www.vaultproject.io/) 📢 **BLOG POST - Comming soon**

## Table of Content

- [Requirements](#requirements)
- [Role Variables](#role-variables)
- - [Default Variables - `defaults/main.yml`](#default-variables--defaultsmainyml)
- - [Required Variables](#defaults-variables--requiredvariables)
- - [Optional Variables](#defaults-variables--optionalvariables)
- [Playbook](#playbook)

## Requirements

- Ansible 2.10+
- Tested on `RHEL`/`RockyLinux` 9 and `Fedora` but should work with compatible distributions.
- Ensure that the `podman` and `loginctl` binaries are present on the target system.
- If the following Ansible collections are not already available in your environment, please install them: `ansible-galaxy collection install ansible.posix` and `ansible-galaxy collection install containers.podman`.

## Role Variables

### Default Variables - `defaults/main.yml`

```yaml
podman_play_user: "{{ ansible_user_id }}"
```
OS user that runs your pod app. If not specified, it uses the user who executed the playbook.

```yaml
podman_play_group: "{{ ansible_user_id }}"
```
OS group for the app user.

```yaml
podman_play_skip_user_and_group_creation: true
```
By default, skip user and group creation. The pod application will run under the user who executed the playbook by default.

```yaml
podman_play_root_dir: "/home/{{ podman_play_user }}/{{ podman_play_pod_name }}"
```
Default application root directory where configuration files, Kubernetes pod YAML definitions, and other directories are stored.

```yaml
podman_play_template_config_dir: "{{ podman_play_root_dir }}/template_configs"
```
Default path where your custom application configs are templated from the `podman_play_custom_conf` variable.

```yaml
podman_play_pod_state: "created"
```
Ensure that the pod is in the created state. The systemd unit ensures that the pod is started. While there is no reason to change this value, modules also support `started` and `absent`.

```yaml
podman_play_pod_recreate: true
```
This ensures that any change in the configuration file or Kubernetes pod YAML definition triggers pod recreation to apply the latest changes, such as an image tag change.

```yaml
podman_play_systemd_no_header: true
```
Do not generate the header, including metadata such as the Podman version and the timestamp, for the systemd file.

```yaml
podman_play_systemd_restart_sec: "10"
```
Configures the time to sleep before restarting a pod systemd unit.

```yaml
podman_play_systemd_restart_policy: "always"
```
Restart policy of the pod systemd unit. Possible options include `no-restart`, `on-success`, `on-failure`, `on-abnormal`, `on-watchdog`, `on-abort`, and `always`.

### Required Variables

The following variables are not set by default, but they are required for deployment. You will need to define these variables. Below are **example** values.

```yaml
podman_play_pod_name: "dashy"
```
Specify your application pod name.

```yaml
podman_play_pod_yaml_definition: |
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      app: "{{ podman_play_pod_name }}"
    name: "{{ podman_play_pod_name }}"
  spec:
    containers:
      - name: "{{ podman_play_pod_name }}"
        image: docker.io/lissy93/dashy:latest
        ports:
          - containerPort: 80
            hostPort: 9500
        stdin: true
        tty: true
        volumeMounts:
          - mountPath: /app/public/conf.yml:Z
            name: dashy_config
    volumes:
      - hostPath:
          path: "{{ podman_play_template_config_dir }}/conf.yml"
          type: File
        name: dashy_config
```
Define the Kubernetes pod YAML definition to be used by the `podman_play` module for deployment. For more details, refer to the [Kubernetes pod documentation](https://kubernetes.io/docs/concepts/workloads/pods/).

### Optional Variables

These optional variables are not required and are not set by default. You can use these variables to extend your deployment.

```yaml
podman_play_custom_conf:
  - filename: "conf.yml"
    raw_content: |
      # Example Raw Config for conf.yml
  - filename: "another_config.conf"
    raw_content: |
      # Example Raw Config for another_config.conf
```
This variable allows you to deploy any number of configurations for your deployment. Content is always templated into the `podman_play_template_config_dir` directory.

```yaml
podman_play_dirs:
  - "{{ podman_play_root_dir }}/var_www_html"
  - "{{ podman_play_root_dir }}/var_lib_mysql"
```
Create additional directories for your application. You can then mount these directories into your pod by defining the paths in the volumes section of `podman_play_pod_yaml_definition`.

```yaml
podman_play_firewalld_expose_ports:
  - "9500/tcp"
```
List of ports in `port/tcp` or `port/udp` format that should be exposed via firewalld.

```yaml
podman_play_pod_authfile: ""
podman_play_pod_build: ""
podman_play_pod_cert_dir: ""
podman_play_pod_configmap: ""
podman_play_pod_context_dir: ""
podman_play_pod_debug: ""
podman_play_pod_executable: ""
podman_play_pod_log_driver: ""
podman_play_pod_log_level: ""
podman_play_pod_network: ""
podman_play_pod_password: ""
podman_play_pod_username: ""
podman_play_pod_quiet: ""
podman_play_pod_seccomp_profile_root: ""
podman_play_pod_tls_verify: ""
podman_play_pod_userns: ""
```
Additional variables related to the `podman_play_module`. Check the [module documentation](https://docs.ansible.com/ansible/latest/collections/containers/podman/podman_play_module.html) for possible values.
With these variables, you can modify pod deployment specifications.


```yaml
podman_play_systemd_after: ""
podman_play_systemd_container_prefix: ""
podman_play_systemd_env: ""
podman_play_systemd_executable: ""
podman_play_systemd_force: ""
podman_play_systemd_pod_prefix: ""
podman_play_systemd_requires: ""
podman_play_systemd_separator: ""
podman_play_systemd_start_timeout: ""
podman_play_systemd_stop_timeout: ""
podman_play_systemd_use_names: ""
podman_play_systemd_wants: ""
```
Additional variables related to the `podman_generate_systemd_module`. Check the [module documentation](https://docs.ansible.com/ansible/latest/collections/containers/podman/podman_generate_systemd_module.html) for possible values.
With these variables, you can modify pod systemd generation.

## Dependencies

No Dependencies.

## Playbook

* Example playbook to deploy your custom container app

```yaml
- name: Manage your pod app
  hosts: yourhost
  gather_facts: true
  roles:
    - role: voidquark.podman_play
```

## License

MIT

## Contribution

Feel free to customize and enhance the role according to your needs. Your feedback and contributions are greatly appreciated. Please open an issue or submit a pull request with any improvements.

## Author Information

Created by [VoidQuark](https://voidquark.com)