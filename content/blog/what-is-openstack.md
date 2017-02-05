+++
type = "post"
featuredpath = ""
date = "2017-02-05T18:51:40Z"
title = "what is openstack"
author = "Sean Mooney"
featured = ""
linktitle = ""
description = "Intro to the OpenStack project"
categories = ["openstack"]
featuredalt = ""
+++

What Is OpenStack?
==================

For those of you who have not heard of "OpenStack" or the <a href="http://openstack.org">OpenStack Foundation</a>,
you might be wondering what it is. Wiki defines it as an a free and open-source
software platform for cloud computing, mostly deployed as an
infrastructure-as-a-service (IaaS).

While that is true, it is incomplete and unless you know what infrastructure 
as a service is its not very helpful.Granted the wiki page does acutlly
explain what openstack and what openstack does but ill have a go anyway.
So OpenStack as software, is a collection of composable inter related projects
primarialy written in python, for creating private and public clouds enviornments.

Openstack provides an abstraction layer that allow a collection of severs 
to be managed as pools of compute, storage and networking services.
compute, storeage and networking are the 3 pillars of IT service that
make up a standed it infractucure. As openstack provides a software interface
to manage these 3 pillars of IT infrstructure it is a Infrastructure as a service offering.


The project that make up Openstacks IaaS offering are what used to be know as
the OpenStack core projects.

Core Projects:
--------------
<ul>
<li> Nova: Compute Project</li>
<li> Neutron: Networking Project</li>
<li> keystone: Autentication Project</li>
<li> Glance: Images as a service Project</li>
<li> Cinder: Block Storeage Project</li>
<li> swift: Object Storage Project</li>
</ul>

Non Core projects?:
-------------------
But if there is a Core then are there more non core projects?
For openstack that is a resounding YES!!! infact since the
creation of the <a href="https://github.com/openstack/governance/blob/master/resolutions/20141202-project-structure-reform-spec.rst">big tent</a> and arguably before, the conpendium of openstack
and openstack releated projects has been expanding beyond IaaS to Orchestration,
Platform as a service, advanced services such as container orchestration and supprot
services such as workflow engings,database as a service, backup as a service and
many other microservices that can be composed to build your own openstack powered application.

A listing of offical project teams can be found in the openstack <a href="https://governance.openstack.org/tc/reference/projects/">govournace</a> repo
and many other affilated projects can be found by exploring the openstack namesace on <a href="https://github.com/openstack">github</a>.


 
How mature are all test projects?
---------------------------------
With all these core and non core project to choose for do i need them all?
How mature are they really? Are they all ready to use in my production application?

Well the answer to the first question is simple, openstack is a compisition of microservices that
work togher to provide a could of your own desigin. if you dont need object storage then you
can deploy without swift, already have ceph deployed? no problem you can have cinder,nova and glance
use that as your storage backend. do you want dns as a service by not orchestration?
you can deploy designate but leave heat out of your cloud deployment.

The secound question of how mature these projects are is a little harder to answer.
The OpenStack Core project have matured over many release to be stable and production
ready, the advanced services are at differing level of maturaty but luckilly the
OpenStack foundation have been developing a tool to help you make up your own mind.

The OpenStack <a href="https://www.openstack.org/software/project-navigator">Project Navigator</a> is a great place to start and if you still
have question the reaching out to the comunity is as easy as jumping on <a href="https://wiki.openstack.org/wiki/IRC">irc</a> or
sending a mail to the <a href="mailto:openstack@lists.openstack.org">mailing list</a>.

Future Reading
-------------- 
<ul>
<li><a href="http://www.openstack.org">Openstack homepage</a></li>
<li><a href="https://www.openstack.org/software">Openstack software overview</a></li>
<li><a href="https://www.youtube.com/watch?v=hWWSaBOMTNo">Openstack architecture 10,000 feet</a></li>
<li><a href="https://www.youtube.com/playlist?list=PLG2eb1MxWbfEqFEbziT9geOOXwiw9zZOm">intro to neutron</a></li>
<li><a href="https://www.youtube.com/user/OpenStackFoundation">Openstack Foundation youtube</a></li>
</ul>
