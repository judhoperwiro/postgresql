
## Ansible Module for installation Postgresql
​ 
#### Prerequisite

- ansible 2.9 
- vim

### 1.) Create Directory for Postgresql 
```shell 
mkdir -p /data/ansible/postgresql/source/
cd /data/ansible/postgresql
vi install-postgresql-12-1.yaml
vi inventory
```
### 2.) Run Ansible Playbook  
```shell 
ansible-playbook -i inventory-<username> install-postgres-12-1.yaml
```

### 3.) Limitiation  
```shell 
Cannot start postgres, database and tablespace using ansible on RHEL7
```
