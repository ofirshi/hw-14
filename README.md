# hw-14

Preparations:
	apt-get install -y sshpass
	cp /etc/ssh/sshd_config{,_orig}
	cat <<EOF > /etc/ssh/sshd_config
	Protocol 2
	HostKey /etc/ssh/ssh_host_rsa_key
	HostKey /etc/ssh/ssh_host_dsa_key
	SyslogFacility AUTHPRIV
	PermitRootLogin yes
	AuthorizedKeysFile      .ssh/authorized_keys
	PasswordAuthentication yes
	ChallengeResponseAuthentication no
	UsePAM yes
	UseDNS no
	AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
	AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
	AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
	AcceptEnv LC_IDENTIFICATION LC_ALL
	Subsystem sftp internal-sftp
	EOF
	systemctl restart sshd

Ansible :
echo "localhost ansible_connection=ssh ansible_ssh_user=root ansible_ssh_pass=screen" >> /etc/ansible/hosts
cat <<EOF > roles/common/tasks/main.yml
	---
	- name: Install packges
	  package:
		name: vim,zip
		state: present
	EOF
	cat <<EOF > ./assignment14.yml
	---
	- hosts: localhost
	  roles:
		- common
	EOF
	export ANSIBLE_DEBUG=1
	ansible-playbook assignment14.yml
	export ANSIBLE_DEBUG=0

Remove:
	ansible servers -m package -a "name=vim,zip state=absent"
	
	
challenge:
	mkdir -p roles/challenge/files
	mkdir -p roles/challenge/handlers
	mkdir -p roles/challenge/tasks

cat <<EOF > roles/challenge/files/hostname.j2
This servers hostname is: {{ansible_hostname}}
EOF
cat <<EOF > roles/challenge/files/test1
Hello world!
EOF
cat <<EOF > roles/challenge/tasks/main.yml
---
- name: Template a file to another place
  template:
    src: ./roles/challenge/files/hostname.j2
    dest: /tmp/test2
- name: text a file to another place
  template:
    src: ./roles/challenge/files/test1
    dest: /tmp/test1
EOF
		cat <<EOF > ./challenge.yml
		---
		- hosts: localhost
		  serial: 1
		  roles:
			- challenge
		EOF

ansible-playbook challenge.yml