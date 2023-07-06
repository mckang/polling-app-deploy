##GCU 2023 Summer

---

#prerequsite

1. install python
2. install ansible
3. KiC Account & API KEY
4. KiC Bastion Host with public ip
5. KiC Bastoin Host key file for jumphost

---

#configuration
secrets/init_values.yml

```
BaseDirectory: 현재 디렉토리 절대경로
TeamName: 팀명

# Variables for ssh config

JumpHost:
Address: Bastion Host 퍼블릭 아이피
User: jumpuser
Port: 22
IdentityFile: "{{ BaseDirectory }}/secrets/{{ TeamName }}-jumpuser"
VPC_NW_Range: 10.0.\*

# Variables for non-ansible vm

DefaultUser: ubuntu
DefaultPrivateKeyFile: "{{ BaseDirectory }}/secrets/manager.pem"

# Variables for ansible config

AnsibleUser: ansible
AnsiblePrivateKeyFile: "{{ BaseDirectory }}/secrets/{{ TeamName }}-ansible"
VM_ANSIBLE_PUB_KEY_FILE: "{{ BaseDirectory }}/secrets/{{ TeamName }}-ansible.pub"

# Variables for kakao i cloud api call

APIAlias: kic
ApplicationCredentialID: KiC Account API Key
ApplicationCredentialSecret: KiC Account API Secret
```

---

#실행
ansible-playbook playbooks/setup.yml

ansible-inventory --list

ansible-playbook playbooks/vms/create.yml

ansible-playbook playbooks/vms/check.yml

ansible-playbook -i playbooks/cluster_inventory.yml playbooks/swarm/playbook.yml

ansible -i playbooks/cluster_inventory.yml -a 'sudo docker node ls' manager

ansible-playbook -i playbooks/cluster_inventory.yml playbooks/deploy/playbook.yml

ansible -i playbooks/cluster_inventory.yml -a 'sudo docker service ls' manager

ansible -i playbooks/cluster_inventory.yml -a 'curl 127.0.0.1' manager

ssh -F ssh.cfg ansible@10.0.3.150 -i ../secrets/team1-ansible
