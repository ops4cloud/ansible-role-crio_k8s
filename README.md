Ansible-role-crio_k8s
=========

This role will deploy by default CRI-O and kubernetes on a bare metal server 

Requirements
------------

- Ansible >= 2.9

Role Variables
--------------

````yaml
# defaults file for ansible-role-crio_k8s
# crio version tested = 1.20 1.21 1.22.1
crio_major_version: "1.22"
crio_minor_version: "1"
crio_os_distro: "{{ ansible_distribution }}"
crio_os_major_version: "{{ ansible_distribution_major_version }}"
crio_cni: flannel
kubernetes_cni: "{{ crio_cni }}"
kubernetes_cni_network: "10.244.0.0/16"
kubernetes_master: False
````

Dependencies
------------

- role: ansible-role-metallb

Example Iventory
----------------

````ini
[all]
localhost ansible_connection=local

[crio:children]
crio_master
crio_worker

[crio_master]
crio-master kubernetes_master=True
[crio_worker]
crio1
crio2
crio3
````

Example Playbook
----------------

````yaml
- hosts: crio
  vars:
    #CNI flannel or calico
    kubernetes_cni: flannel
  roles:
    - 
      { role: ansible-role-crio_k8s }
    -
      { role: ansible-role-metallb, when: kubernetes_master }
  post_tasks:
    - name: pull admin config for local kubectl
      synchronize:
        mode: pull
        src: ~/.kube/config
        dest: __$HOME_CHANGE_ME__/.kube/config
      when: kubernetes_master

- hosts: crio_worker
  tasks:
    - name: join cluster
      shell: "{{ hostvars['crio-master'].join_command }} >> node_joined.txt"
      args:
        chdir: $HOME
        creates: node_joined.txt
````


License
-------

BSD

Author Information
------------------

gotoole
ops4cloud.fr