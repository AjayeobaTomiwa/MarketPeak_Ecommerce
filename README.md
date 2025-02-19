# MARKETPEAK ECOMMERCE PRROXY SERVER DEPLOYMENT

## Project Details

This Project aimed to achieve a static web hosting on an EC2 Web server, using proxy server deployment. It aims to achieve Git version control, local web deployment in a linux environment, Continuous integration and Deployment. This `README.md` markdown file helps to document the process of achieving this process, problems encountered, troubleshooting solutions applied and final results. To achieve proper documentation, this `README.md` file has been sectioned into six.

## Section 1 - Sourcing a website template

I checked a template pool sourced from the [internet](https://www.tooplate.com/), to source for a suitable static website that included `html, css, and Javascript` files of the static
website.

> Next, i downloaded the zip file of the template website, and extracted it into a folder named `Files`

> In order to have a website that follows the ecommerce theme, made changes to the `index.html` file of the template using vscode, changed the texts, modified the headings and body to suite the ecommerce theme. once i was done, i saved changes.

## Section 2 - Creating a git repository

Next i opened git bash and changged directory to my preferre directory location:

```
cd /home/tomiw/
```

Next, i created a new directory using

```
mkdir MarketPeak_Ecommerce
```

changed directory to the new directory created

```
cd /home/tomiw/MarketPeak_Ecommerce
```

Now inside the new directory, i initialized a new git repository using:

```
git init
```

i moved the website template from its original downloaded loaction to the repository directory using:

```
mv /home/tomiw/Downloads/Files /home/tomiw/MarketPeak_Ecommerce
```

next i added the repository to the local workflow using:

```
git add .
```

and committed the directory using

```
git commit -m "Commit dev files of marketpeak to new repository"
```

Next i pushed the git repository using

```
git remote add origin https://github.com/AjayeobaTomiwa/MarketPeak_Ecommerce.git
```

Upnext i pushed the code using:

```
git push -u origin main
```

## Section 3 AWS EC2 Instance Creation

On the AWS LOGIN console i signed in as `IAM User` called `Chyliper-v1` which is under the `Developers` IAM Group that has been earlier createed.

> Next i switched to the nearest availability zone located in South Africa named `af-south-1`

> Next i navigated to EC2 using the search bar on AWS.

> Next i created an EC2 Instance using the following conditions:

```
Operating system: Ubuntu Server 24.04 LTS (HVM) SSD Volume Type (free tier eligible)

Instance type: t3 micro

Storage(volume): 8Gb

Key pair: Existing key pair (ubuntu.pem)

security group: existing security group(MarketPeakSg)
```

### Security group credentials

---

On AWS, I made modified changes to the inbound rules of the `MarketPeakSg` security group. The idea was to make the instance attached to the security froup accessiblle by none other asides my local machine which is best practice. Therefore i followed the following steps:

> copied the gateway IP address of my local machine by inputing the following cmd on windows powershell:

```
ipconfig
```

> this brought out all existing ip addresses (ethernet, gateway and wi-fi), and i just had to take note of the gateway ip address.

Next up, in my inbound rules, i added a rule giving ssh access to only my local machine's `<gateway/ip-adress>` on port 22.

Added inbound rules to support all http entries on port 80,and saved the changes.

Up next, i opened mobaXterm and tried to login using my existing `ubuntu.pem` key pair which already had the appropriate read and write permissions. using the following command to establish an ssh connecction to the EC2 iNSTANCE.

```
ssh -i "ubuntu.pem" ubuntu@ec2-13-61-214-198.eu-north-1.compute.amazonaws.com
```

I successfully gained entry into the Ec2 Instance. as `ubuntu@<public Ip>`

## Section 4 - Connecting EC2 instance with my Git Repository

To do this i needed to establish ssh key connection with my EC2 instance with my git account.

> First, I gained access as a root user using

```
sudo su
```

> Next i updated the package installer `aot`

```
apt update -y
```

next, after successfully updating apt, i generated a new ssh-key

```
ssh-keygen
```

> i saved the key gen files as `id_rsa.pub` and `id_rsa`, resoectfully.

> next, i copied the ssh public key from the `id_rsa` file and used it to create a new ssh key on github. Naming the ssh key `newsshKey`

next, i tried to check if my ssh key is being recognized by inputing

```
cat /home/ubuntu/.ssh/id_rsa.pub

Result:
cat: /home/ubuntu/.ssh/ssh: No such file or directory
cat: key_gen.pub: No such file or directory
```

This meant there was a problem somewhere, but regardless, i went ahead to connect the key with my instance using

```
git clone git@github.com:AjayeobaTomiwa/MarketPeak_Ecommerce.git

Result:
Cloning into 'MarketPeak_Ecommerce'...
The authenticity of host 'github.com (4.225.11.194)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes\
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

The connection was not made, so i had to run some troubleshooting:

### Troubleshooting

to find the root of my problem, i had to ensure the id_rsa file existed using :

```
ls -l id_rsa.pub
```

indeed, the file was present.

Next, i needed to check if the key credentials were actually present using:

```
cat /home/ubuntu/.ssh/authorized_keys
```

indeed the ssh key was present, then i went researching. After extensive researching for the connectivity error possibilities. i figured out that there are actually two types of ssh key connections; `RSA` and `ED_ 25519` connections. i quickly understood that for a secure connection to be successfully established, either `RSA` or `ED_ 25519` connections must be made on both sides i.e. the github ssh key and the instance id.pub key. having either one on either side will result in an error in connection. To fix this;

- I changed the file name of `id_rsa` and `id_rsa.pub`to `id_ed25519` and `id_ed25519.pub` respectively.
- discovered that the initial ssh key generated was that of `ED25519` using:

```
mv ~/.ssh/id_rsa ~/.ssh/id_ed25519
mv ~/.ssh/id_rsa.pub ~/.ssh/id_ed25519.pub
```

- checked if the `id_ed25519` still contained the ssh key using

```
cat ~/.ssh/id_ed25519
```

- copied the ed-25519 key again to github and ran the ssh authentication command again

```
ssh -T git@github.com

Result: Hi AjayeobaTomiwa! You've successfully authenticated, but GitHub does not provide shell access.
```

- afterwards i could proceed to clone my repository using ssh path

```
git clone git@github.com:AjayeobaTomiwa/MarketPeak_Ecommerce.git

Result: Cloning into 'MarketPeak_Ecommerce'...
remote: Enumerating objects: 36, done.
remote: Counting objects: 100% (36/36), done.
remote: Compressing objects: 100% (34/34), done.
remote: Total 36 (delta 1), reused 36 (delta 1), pack-reused 0 (from 0)
Receiving objects: 100% (36/36), 2.46 MiB | 3.92 MiB/s, done.
Resolving deltas: 100% (1/1), done.
root@ip-172-31-31-218:/home/ubuntu# apt install htmld -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package htmld
root@ip-172-31-31-218:/home/ubuntu# yum install htmld -y
Command 'yum' not found, did you mean:
  command 'gum' from snap gum (0.13.0)
  command 'uum' from deb freewnn-jserver (1.1.1~a021+cvs20130302-7build1)
  command 'sum' from deb coreutils (9.4-2ubuntu2)
  command 'num' from deb quickcal (2.4-1)
  command 'zum' from deb perforate (1.2-5.3)
See 'snap info <snapname>' for additional versions.
```

and just like that, problem solved.

## Activating Reverse Proxy using Apache2

Initially, when i saw the capstorne project guide instructing me to use `yum` to install `apache2` as `httpd`, i knew straight away from previous knowledge that yum is not a linux command. Instead i used apt to install apache2 while still on the root user

```
apt install apache2 -y
```

after apache was successfully installed, i proceeded to replace the apache recognized html file for static web hosting in the path `/var/www/html/*` with my MarketPeak Repository directory using the following cmds:

```
rm -rf /var/www/html/*
cp -r /home/ubuntu/MarketPeak_Ecommerce/* /var/www/html
sudo systemctl reload apache2
```

By now the static webpage was visible after loading the EC2instance's public ip address to my web browser.

## Section 5 - Making changes to the static website

Back to vscode, i created a new branch on the VSCode terminal called `Development`

```
git checkout -b Development
```

made changes to the `index.html` file,saved the changes, added the changes to the working tree, committed the changes, and created a pull request to the main branch using the following commands:

```
git add .
git status
git commit -m "Changes made to Home page, "Past Projeccts" and "Background" tabs added"
git push origin Development
```

On github, using the pull request made by the Development branch, i merged the changes with the main branch and confirmed changes.

back to the terminal, i switched to the main branch, pulled the updates and changes to the main branch using

```
git checkout main
git pull origin main
```

Back to the Linux Ubuntu Machine, i changed directory to the repository located on the apache web hosted location.

```
cd /var/www/html/MarketPeak_Ecommerce
```

pulled the updated changes using

```
git pull origin main
```

updated the apache2 proxy server using

```
sudo systemctl reload apache2
```

refreshed the web browser where the IP ADDRESS of the instance is loaded and saw the updated changes.

## Section 6 - Uploading the `README.md` file

Back to VSCode Editor, i made changes to the `README.md` file. After saving the changes, i added, committed and pushed the changes made to the main branch using the following commands:

```
git add README.md
git commit -m "README.md file update"
git push origin main
```

On github, i created a pull request, merged the changes and saved the changes. END OF PROJECT.

## Final Result

The aim of the project to achieve Git version control, local web deployment in a linux environment, Continuous integration and Deployment of a static web page was fulfilled as i learned the intricate aspects of deploying a static web page using reverse proxy (Apache2), CI/CD operation, and git version control.
