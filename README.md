# PIC-SURE all-in-one build with intergated database authentication

Assumptions:

- This system will be maintained by someone with either a basic understanding of Docker or the will to learn and develop that understanding over time.

- The server can access the internet and your browser can access the server on ports 80, 443, 8080

- You have sudo privileges or root account access on the server.

Preparing to deploy:

You will be creating an initial admin user tied to an eamil account.  A default password will be created with the value 'password'.

Before you can safely run the system in production you will need an SSL certificate, chain and key that is compatible with Apache HTTPD. If you are unable to obtain secure SSL certs and key, and are taking steps to keep your system from being accessible to the public internet you can choose to accept the risk that someone may steal your data or hijack your server by using the development certs and key that come installed by default. -- *USE THE DEFAULT CERTS AND KEY AT YOUR OWN RISK* --


Minimum System Requirements:

- 32 GB of RAM
- 8 cores
- 100 GB of hard drive space plus enough to hold your data
- Only Centos 7 is supported for operating systems

# Steps to install on a fresh Centos 7 installation:

- Install Git

sudo yum -y install git

- Clone the PIC-SURE All-in-one-db-auth repository (this repo)

git clone https://github.com/hms-dbmi/pic-sure-all-in-one-db-auth

- Install the dependencies and build the Jenkins container.  This process will start jenkins automatically.

cd pic-sure-all-in-one/initial-configuration

sudo ./install-dependencies.sh


- Browse to Jenkins server

Point your browser at your server's IP on port 8080. Work with your local IT department to make sure that this port is not available to the public internet, but is accessible to you when on your intranet or VPN. Anyone with access to this port can launch any application they wish on your server.

If your server has IP 10.109.190.146, you would browse to http://10.109.190.146:8080

In Jenkins you will see 5 tabs: All, Configuration, Deployment, PIC-SURE Builds, Supporting Jobs

On the Configuration tab click the button to the right of the Initial Configuration Pipeline job. It looks something like a clock with a green triangle on it. You will then be asked for the following information:

EMAIL - This is the email address that will be the initial admin user.

PROJECT_SPECIFIC_OVERRIDE_REPOSITORY - This is the repo that contains the project specific overrides for your project.  For a sample, see https://github.com/hms-dbmi/baseline-db-auth-pic-sure

RELEASE_CONTROL_REPOSITORY - This is the repo that contains the build-spec.json file for your project. This file controls what code is built and deployed.  For a sample, see https://github.com/hms-dbmi/baseline-pic-sure-release-control

Make sure none of these fields have any leading or trailing whitespace, the values must be exact. Once you have entered the information, click the "Build" button.

Wait until all jobs complete, it will take at least several minutes. When there is nothing showing in the Build Queue or Build Executor Status to the left of the page, all jobs will have completed.

Check the All tab to make sure nothing shows with a red dot next to it. If you see any red dots try starting from scratch with a fresh Centos7 install. If you consistently have the same job(s) fail(red dots) then you should reach out to avillach_lab_developers@googlegroups.com for help.

If all jobs have blue dots except the Check For Updates and Configure SSL Certificates job, which should be gray, you should be able to log into the UI for the first time. Browse to the same domain or IP address as your Jenkins server, but without the 8080 port.

If your server has IP 10.109.190.146, you would browse to https://10.109.190.146

Log in using your Google account that you configured in step 5 above.

Once you have confirmed that you can access the PIC-SURE UI using your admin user, stop the jenkins server by runnning the stop-jenkins.sh script:

sudo ./stop-jenkins.sh

Any time you wish to update the system in any way you will need to run the update-jenkins.sh script, this will make sure the jenkins jobs and configurations are up to date and then start the Jenkins server. You should always stop Jenkins using the stop-jenkins.sh script when you are done to prevent unauthorized access as Jenkins effectively has root privileges on your server.

To start or stop PIC-SURE use the "Start PIC-SURE" and "Stop PIC-SURE" jobs.

To start or stop JupyterHub use the "Start JupyterHub" and "Stop JupyterHub" jobs. The Start JupyterHub job will ask you to set a password. Currently this password is shared by all JupyterHub users, we are working to integrate JupyterHub with the PIC-SURE Auth Micro-App so that users can log in using the same credentials they use to access PIC-SURE UI. To access JupyterHub browse to your server ip address on the path /jupyterhub

If your server has IP 10.109.190.146, you would browse to https://10.109.190.146/jupyterhub

If you have Apache HTTPD compatible certificate, chain and key files for SSL configuration, go to the Configuration tab and run the Configure SSL Certificates job uploading your server.crt, server.chain and server.key files using the Choose File buttons, then hit the Build button. Once this completes, go to the Deployment tab and run the Deploy PIC-SURE job to restart your containers so the updated SSL configuration is used.

As your project progresses you will run the Check For Updates job to pull and build the latest release of each component as the release control repository is updated. To deploy the latest updates after Check For Updates is run, execute the Start PIC-SURE job.

