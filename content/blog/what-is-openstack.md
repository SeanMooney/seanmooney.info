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
as a service is it's not very helpful.Granted the wiki page does actually
explain what OpenStack and what OpenStack does but ill have a go anyway.
So OpenStack as software, is a collection of composable interrelated projects
primarily written in python, for creating private and public clouds environments.

OpenStack provides an abstraction layer that allows a collection of servers 
to be managed as pools of compute, storage and networking services.
compute, storage and networking are the 3 pillars of IT service that
make up a standard IT infrastructure. As OpenStack provides a software interface
to manage these 3 pillars of IT infrastructure it is an Infrastructure as a service offering.


The project that makes up OpenStack IaaS offering are what used to be know as
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

Non-Core projects?:
-------------------
But if there is a Core then are there more non-core projects?
For OpenStack that is a resounding YES!!! in fact since the
creation of the <a href="https://github.com/openstack/governance/blob/master/resolutions/20141202-project-structure-reform-spec.rst">big tent</a> and arguably before, the compendium of OpenStack
and OpenStack related projects has been expanding beyond IaaS to Orchestration,
Platform as a service, advanced services such as container orchestration and support
services such as workflow engines, database as a service, backup as a service and
many other micro services that can be composed to build your own OpenStack-powered application.

A listing of offical project teams can be found in the openstack <a href="https://governance.openstack.org/tc/reference/projects/">govournace</a> repo
and many other affilated projects can be found by exploring the openstack namesace on <a href="https://github.com/openstack">github</a>.


 
How mature are all test projects?
---------------------------------
With all these core and non-core project to choose for do I need them all?
How mature are they really? Are they all ready to use in my production application?

Well, the answer to the first question is simple, OpenStack is a composition of micro services that
work together to provide a could of your own design. if you don't need object storage then you
can deploy without swift, already have ceph deployed? no problem you can have cinder,nova and glance
use that as your storage backend. do you want DNS as a service by not orchestration?
you can deploy designate but leave heat out of your cloud deployment.

The second question of how mature these projects are is a little harder to answer.
The OpenStack Core project have matured over many releases to be stable and production
ready, the advanced services are at a differing level of maturity but luckily the
OpenStack foundation has been developing a tool to help you make up your own mind.

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
