---
title: ansible example
tags: ansible
grammar_cjkRuby: true
---

###  deploy.sh 
```shell
#!/bin/sh
host=$3
if [ ! -n "$1" ]; then
	echo
	echo "=====================Usage============="
	echo "Usage:$0 playbook_name commit_id [hosts]"
	echo "           $0 playbook_name -c [hosts] to check deploy status"
	echo"            $0 playbook_name -l  [hosts] to list hosts"
	echo"            $0 playbook_name -s [hosts] to check syntax"
	echo"            $0 playbook_name -start  [hosts] to start app"
	echo"            $0 playbook_name -stop  [hosts] to stop app"
	echo"            $0 playbook_name -restart  [hosts] to restart app"
	echo"======================End=============="
	exit 1
fi

if [ ! -n "$2" ]; then
	echo
	echo "=====================Usage============="
	echo "Usage:$0 playbook_name commit_id [hosts]"
	echo "           $0 playbook_name -c [hosts] to check deploy status"
	echo"            $0 playbook_name -l  [hosts] to list hosts"
	echo"            $0 playbook_name -s [hosts] to check syntax"
	echo"            $0 playbook_name -start  [hosts] to start app"
	echo"            $0 playbook_name -stop  [hosts] to stop app"
	echo"            $0 playbook_name -restart  [hosts] to restart app"
	echo"======================End=============="
	exit 1
fi

if [ ! -n "$3" ]; then
	echo "use default hosts:all"
	host=all
fi

cd /home/test/playbooks/$1/app-deploy
source /tools/ansible/bin/activate

if [ "$2" == "-l" ]; then
	ansible-playbook -i hosts site.yml --list-hosts -e "hosts=$host"
	exit 0
fi

if [ "$2" == "-c" ]; then
	ansible-playbook -i hosts $host -m script -a "check.sh"  -u test
	exit 0
fi

if [ "$2" == "-s" ]; then
	ansible-playbook -i hosts site.yml --syntax-check -e "hosts=$host"
	exit 0
fi

if [ "$2" == "-d" ]; then
	ansible-playbook -i hosts -v -k deliver_ssh_key.yml --extra-vars "hosts=$host"
	exit 0
fi

if [ "$2" == "-start" ]; then
	ansible-playbook -i hosts -v -k start.yml --extra-vars "hosts=$host"
	exit 0
fi

if [ "$2" == "-stop" ]; then
	ansible-playbook -i hosts -v -k stop.yml --extra-vars "hosts=$host"
	exit 0
fi

if [ "$2" == "-restart" ]; then
	ansible-playbook -i hosts -v -k restart.yml --extra-vars "hosts=$host"
	exit 0
fi
ansible-playbook -i hosts -v site.yml --extra-vars "commit_id=$2 hosts=$host"
```

### site.yml
```yaml
- hosts:app
- roles:
	- role: deploy
```

### check.sh
```shell
#!/bin/sh

#check pid
pids=$(ps ux |grep "/tools/apps/test_dubbo" |grep -v grep |awk {'print $2'})
if [ "$pids"a == "a" ]; then
	echo "Progress is not exist."
	exit 1
fi

#check port
portFlag=`netstat -an |grep 20880 |grep LISTEN wc -l`
if [ "$portFlag" != "1" ]; then
	echo "Port:20880 is not avalibe."
	exit 1
fi

#check log
startedFlag=`cat /data/appLogs/test.log| grep "dubbo service has started" | wc -l`
if [ "$startedFlag" != "0" ]; then
	echo "log is not fund"
	exit 1
fi

echo "SUCCESS"
```

###  deliver_ssh_key.yml
```yaml
- hosts: app

tasks:
	- name:deliver authorized_keys
	  authorized_key:
	  		user: nucc
			key: "{{lookup('file','/home/test/.ssh/id_rsa.pub')}}"
			state: present
			exclusive: no

```

### start.yml
```yaml
- hosts: app
   
   tasks:
   		- name: 启动jetty容器
   		  shell:  nohub /tools/jetty/bin/jetty.sh start
```

### stop.yml
```yaml
- hosts: app
   
   tasks:
   		- name: 停止jetty容器
   		  shell:  nohub /tools/jetty/bin/jetty.sh stop
```

### restart.yml
```yaml
- hosts: app
   
   tasks:
    	- name: 停止jetty容器
   		  shell:  nohub /tools/jetty/bin/jetty.sh stop
   		- name: 启动jetty容器
   		  shell:  nohub /tools/jetty/bin/jetty.sh start
```