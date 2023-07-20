##GCU 2023 Summer

---

#prerequsite

1. install python
2. install ansible
3. KiC Account & API KEY
4. KiC Bastion Host with public ip
5. KiC Bastoin Host key file for jumphost

6. 공공클라우드 활용에 따른 변경 사항
   https://account.kakaoicloud-kr-gov.com/domain
   
   gcu-master 키페어 생성
   gcu-vm 보안그룹 생성 : bastion 보안그룹과의 통신 설정, lb 와의 통신 설정
   ```
   templates/clouds.yml
   region_name: 'kr-gov-central-1'
   auth_url: 'https://iam.kakaoicloud-kr-gov.com/identity/v3'
   ```
   ```
   ansible/playbooks/vms/vars.yml
   subnet_id: bd6d7c3b-1784-4d9e-92d5-1fb939cb555c
   VM_FLAVOR_NAME: m1i.xlarge
   ```   
   ```
   ansible/playbooks/vms/create.yml
   VM_KEY_NAME: "gcu-master"
   VM_SG_NAMES: 
     - "gcu-vm"
   ```
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

기초 환경 설정
ansible-playbook playbooks/setup.yml

KiC API 연결 확인
ansible-inventory --list

VM 생성 
ansible-playbook playbooks/vms/create.yml

VM 생성 완료 확인
ansible-playbook playbooks/vms/check.yml

Docker Swarm Cluster 구성
ansible-playbook -i playbooks/cluster_inventory.yml playbooks/swarm/playbook.yml

Docker Swarm Cluster 확인
ansible -i playbooks/cluster_inventory.yml -a 'sudo docker node ls' manager

서비스 배포
ansible-playbook -i playbooks/cluster_inventory.yml playbooks/deploy/playbook.yml

서비스 배포 확인
ansible -i playbooks/cluster_inventory.yml -a 'sudo docker service ls' manager

서비스 동작 확인
ansible -i playbooks/cluster_inventory.yml -a 'curl 127.0.0.1' manager

VM 접속
ssh -F ssh.cfg ansible@VM 아이피 -i ../secrets/team1-ansible
