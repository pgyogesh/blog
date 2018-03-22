---
layout: post
title:  "Making MarkLogic cluster highly available"
date:   2017-11-04 01:30:13 +0800
categories: MarkLogic
read: 10 minutes read
tags: marklogic NoSQL High-availability,
---

Marklogic requires minimum three hosts for high availability as it works on qourum and I had three virtual machines(Say, host_1,host_2 and host_3) to check MarkLogic high a availability.

I'll not be explaining installation and adding hosts to marklogic cluster as docs.marklogic.com has guides to do the same.

Here are links to those guides:

- [Installing MarkLogic](http://docs.marklogic.com/guide/installation/procedures#id_28962)


- [Adding host to cluster](https://docs.marklogic.com/guide/cluster/config_cluster#id_65834)

Once you added the hosts to the cluster, you will not see defaults forests such as Security, Schema, App-services etc. on newly added hosts. So even if you have three hosts in cluster and first initialized host is down then you will not able to access MarkLogic as it requires Security database data to authenticate users and App-services database data to access the app servers and so on.

So, you need to create replica forests on newly added hosts. And here is how we can create replica forests for security forest and make MarkLogic highly available. You can use same procedure for other schemas.

`
Note: You can't create replica of private forests and all default forests are private.
`

1. Create one public forest security1 in host_1.

    ![Security2](https://raw.githubusercontent.com/pgyogesh/blog/master/_posts/images/security1.png "Security1")

2. Create one more public forest security2 in host_2.

    ![Security2](https://raw.githubusercontent.com/pgyogesh/blog/master/_posts/images/security2.png "Security2")


3. Create another public forest security3 in host_3.

`Note: Directory path that you are providing while creating forest should be present in hosts.`


Now attach security1 forest to Security database using below steps.

- In admin console go to:
      [+] Configure
          [-] Forests

- In right panel, Attach security1 to security database.

  ![Attaching forest to database](https://raw.githubusercontent.com/pgyogesh/blog/master/_posts/images/security3.png "Attaching forest to database")

- Make security2 and security3 replica of security1

      [+] Configure
          [-] Forests
              [-] Security1

  In right panel, Click on configure and add replica to security1
  ![Adding replica](https://raw.githubusercontent.com/pgyogesh/blog/master/_posts/images/security4.png "Adding replica")

- Retire default security forest from security database.

      [+] Configure
          [-] Databases
              [-] Security
                  [-] Forests

   In right panel, select retired checkbox of Security forest and click OK. This process will copy data from Security forest to security1 forest.
   `Note : Do not detach and retire in single step`

   ![Retiring forest](https://raw.githubusercontent.com/pgyogesh/blog/master/_posts/images/security6.png "Retiring forest")

- Once security forest is retired, You can detach security forest from security database.

   ![Detaching forest](https://raw.githubusercontent.com/pgyogesh/blog/master/_posts/images/security6.png "Detaching forest")

Also, you must follow same process for other forests such as **App-services, schema** to access those from all the nodes in MarkLogic.
