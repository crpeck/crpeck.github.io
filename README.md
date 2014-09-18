##Materials for MABUG 2014 Presentation

###Title: Automated Build and Deployment of Tomcat Apps, ~~or, How Ops can get out of the way of development~~

* Chris Peck
* College of William and Mary
* Date: 9/29/2014
* Session ID: 8743


####Intro
Last Fall, the IT Unix "Ops" team at William and Mary implemented a new tool named Jenkins for managing software projects.  Eventually, we were able to to talk some Developers into using this tool for the development of their Grails Apps.  This presentation will describe the journey to a faster, less-painful deployment process for Grails (or any other) Apps to a Tomcat server.  I will describe the processes in place before, how we got to where we are, and the tools we used to implement the current environment.

####Note
After I submitted th the "final" version of this presentation to MABUG, our team was informed that Banner XE would be deployed to Linux servers (which is what our team “does”).  I had attended MABUG last Fall to learn more about what XE was, the development strategy and had read how Ellucian appeared to be using technologies similar to what we had focused on using as we began to rollout our “next-generation” of servers & services in *nixland*. I was intrigued to say the least.  I had a talk over some beers at the pig roast with Ed Hauser & a “tech” person from Ellucian, and, we seemed in sync conceptually on this. We are planning on using the same processes described here to deploy Banner XE Apps as they become available.

####Pre-Jenkins
In the pre-Jenkins world, Grails warfiles were built on the developers desktop, then copied to a network "drive". They would then contact the Ops team who would copy it from the network drive to the Tomcat server, put it in place and restart the service. This led to a lot of pain points, what version of grails and java was being used, etc. The age-old "Well, it worked on my Desktop" statement was rather common. Much of this process was manual, making the whole process fragile, with many points of possible failure. Since Developers depended on Ops for deployment it was a very slow process
#####Issues
* warfile built on Dev's Desktop, using their version of Grails and Java
* deployment was manual, copying the file from a shared Network *drive*. Ops had to get the correct warfile to the correct server. Required restarting Tomcat to re-deploy.
* slow process to deploy the App

####Post-Jenkins
In the post-Jenkins world, well, Jenkins does it all for us, with some help from some other tools.  When code is committed to our local source code repository, Jenkins will automatically build the War files (in the same environment as the Tomcat servers).  If Jenkins can build the App successfully, the following occurs:
For the Development Environment the WAR file is automatically deployed to the Tomcat server. 
For the Production Environment, since Ops are the only ones allowed to deploy to PROD, then someone from Ops tells Jenkins to deploy the App. 
Since the process is automated, the chance of failure is minimized. Developers are happy, and, Ops are ecstatic.


####Demonstration
During the creation of this presentation, I created a "Vagrantfile" for demonstrating how this is done. This Vagrantfile will spin up an Ubuntu 14.04 server VM running Tomcat and Jenkins. It also installs a simple source tree using git to use as an example. This git source tree contains a post-commit hook into Jenkins to kick off the building of a warfile based on it, and, then auto-reploying it to the Tomcat server.

If you would like to use this Vagrantfile to play with, head on over to https://github.com/crpeck/vagrant-mabug and *git* it.

I also created a real simple sourcetree that prints out *hello world* that get's installed into /usr/local/src/hello-world on the Vagrantbox VM. This is the source tree mentioned above, it's located at https://github.com/crpeck/hello-world

####Running the Demonstration
Running through this assumes the following:
* You've installed VirtualBox
* You've installed Vagrant
* You've installed git
Once you have those requirements out of the way, here's a quick walk through.
```
Open a terminal window
mkdir mabugdemo
cd mabugdemo
git clone https://github.com/crpeck/vagrant-mabug.git
cd vagrant-mabug
vagrant up

Assuming all went as planned, you should see this as Vagrant brings up the machine - before the "Provisioning" section:
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 8080 => 8080 (adapter 1)
    default: 8090 => 8090 (adapter 1) 
    default: 22 => NNNN (adapter 1) (the NNNN doesn't matter)

and, this at the end:

Jenkins is available at http://localhost:8090
Tomcat manager at http://localhost:8080/manager
  user=manager, password=12345
```

At this point if you bring up a browser and point it at http://localhost:8090 you should see the Jenkins Main Screeen.
The first thing we need to do is build the hello-world warfile and install it in Tomcat.  Go ahead and Click on the *hello-world* listed under Name in the middle of the screen.
Click on *Build Now* in the Left Column, this will build the warfile and install it. Assuming it went ok (you do have a Blue Ball and not a Red one, right?  Click on http://localhost:8080 and you should see:
```
It works !

Click here to see the hello-world example
```

#####Testing the git-commit hook
To test that the post commit hook from git to Jenkins is working, you will need to ssh into the VagrantBox and edit the file that displays Hello World. From the terminal window used to do a *vagrant up* in:
~~~
vagrant ssh
cd /usr/local/src/hello-world/webapp
vi index.jsp
change Hello World to something else
:wq
git commit -am 'changed output to something else'

At this point the Jenkins page should show a Build Job kicking off for the hello-world project (bottom of the left column).  Note that you can display the console output of the build process by clicking on the Job and selecting Console Output. Once it's finished, if you refresh the Hello World browser window it should display the *something else* you entered above.

Here's the contents of the post commit hook:
```
#!/bin/bash
# post commit hook to ping Jenkins to rebuild & redeploy
curl -X POST http://localhost:8090/job/hello-world/build
```

Just a simple curl to ping Jenkins that it needs to build the hello-world project.  You're git server should have a configuration setting where you can do the same thing, we use http://gitlab.com as our local gitserver.

####Links to Tools Used
* Jenkins https://jenkins-ci.org
* git https://git-scm.org
* Vagrant https://www.vagrantup.com 
* VirtualBox https://www.virtualbox.org
* Puppet: http://puppetlabs.com Configuration Management
* Packer http://www.packer.io/ We use it to build our VM images

####Links to Other Great Stuff
* U.S. Digital Services Playbook https://playbook.cio.gov/
* DevOps http://devops.com
* FlowCon Conference http://http://flowcon.org/
* O'Reilly Conference http://velocityconf.com/
* IT Revolution Press http://itrevolution.com/

####Written Material aka Books
* The Phoenix Project: A Novel About IT, DevOps, and Helping Your Business Win, Gene Kim, Kevin Behr, George Spafford
* Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation, Jez Humble and David Farley

####Links to Ellucian Sites
* Demo of Using Ellucian's git repo http://udca.sungardhe.com/cps/WBT/XE_Webinars/using_git_repos_screencast.mov
