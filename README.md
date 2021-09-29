# This allows you to establish a tunnel from your machine into google colab and code locally!
# Hopefully a better UI and code completion
# Additionally run other codes than PY :) 

## Mount Drive


```python
from google.colab import drive
drive.mount("/content/gdrive")
```

## Start a TCP Tunnel and Create SSH Command and Passwd


```python
# Import Dependencies
import random, string, json, getpass
from IPython.display import clear_output
```


```python
#Generate root password
password = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(20))

#Download ngrok
!wget -q -c -nc https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
!unzip -qq -n ngrok-stable-linux-amd64.zip

#Setup sshd
! apt-get install -qq -o=Dpkg::Use-Pty=0 openssh-server pwgen > /dev/null

#Set root password
!echo root:$password | sudo chpasswd
!mkdir -p /var/run/sshd
!echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
!echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
!echo "LD_LIBRARY_PATH=/usr/lib64-nvidia" >> /root/.bashrc
!echo "export LD_LIBRARY_PATH" >> /root/.bashrc

#Run sshd
get_ipython().system_raw('/usr/sbin/sshd -D &')

#Ask for Token
print("Copy AuthToken from https://dashboard.ngrok.com/auth")
authtoken = getpass.getpass()

#Create tunnel
get_ipython().system_raw('./ngrok authtoken $authtoken && ./ngrok tcp 22 &')

#Get Public Address and Save into a json
!curl -s http://localhost:4040/api/tunnels | python -m json.tool > tmp.json

# Read the URL and create Command
f = open('tmp.json',)
url = json.load(f)['tunnels'][0]['public_url'].split(':')

dispStr = f'''
\n************************************Root PassWord***************************\n
                                   {password}
\n************************************Lovely SSH Command**********************\n
                             ssh root@{url[1][2:]} -p {url[2]}\n\n'''

clear_output(wait=True)
print(dispStr)
f.close()

# Remove Unnecessary Files
!ls | grep -v gdrive | xargs rm -r
```

## Config the Thing for ySelf


```python
!sudo apt-get update -qq
!sudo apt-get dist-upgrade -yy
!sudo apt-get autoremove -yy
!sudo apt-get autoclean -yy

!sudo apt-get install git vim tree tmux gcc -y

!alias l="ls -ltrha"
!alias deleteAll="ls | sudo xargs rm -r"
```
