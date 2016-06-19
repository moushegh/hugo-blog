---
title: how to connect windows 2008R2 to windows 2003R2 Domain controller
author: naut
layout: post
date: 2010-05-03
url: /post/how-to-connect-windows-2008r2-to-windows-2003r2-domain-controller/
categories:
  - how to
  - interesting
tags:
  - ad
  - domain controller
  - RODC
  - windows 2003R2
  - windows 2008R2

---
Few days ago i get a chance to configure new windows 2008R2 on already working infrastructure with windows2003R2,
  
main goad was to create RODC (read only domain controller) within AD.
  
after starting the project i get few real interesting situations that want to share.

<!--more-->1. we need windows 2003R2 with domain controller role on it.

2. we need to install windows 2008R2 on separate machine.

after we need to connect windows 2008R2 to our infrastracture as usual host. login on it with domain administrator account and install DNS role.

after DNS is installed we need to run DCPROMO (it will install all nessary roles for domain conntroler). seems everything easy but after few next&#8217;s win2008 will tell you that you need to prepair windows2003 for migration

what we need to do in this case.

1. we need to put win2008 disk to win2003 machine. and copy directory (in support) adprep to C:\ ,

after we need to go to C:\adprep (in CMD) and run one for the file ( for your machine ARCH) for me it was 64 bit version

command is

adprep.exe /forestPrep

it will tell you that its going to update schema  of  2003 to newer version

NOW TIP , you need to PUSH &#8220;C&#8221; and only after ENTER button, in other case it will give you

0x4 error :))

after pushing C and ENTER it will work on your domain like 3-4 minutes. and  will report you that domain is ready for migration

but you need to run here one more command

adprep /domainPrep,

if it will work fine means that your DC working on NATIVE mode if not you need to go to Acitve directory and user settings and RISE your domain till 2003 version.

after RISING domain version you need to run one more time /domainPrep command. and it will finish without problems.

now we need to rin adprep /rodcprep it will make RODC.

after all this steps done, you can return to windows2008 and finish DCPROMO steps.

one restart and your windows 2008R2 is in working infrastructure of windows 2003r2 as secondary domain controller.

one more tip &#8211; if errors will continue apper &#8211; try adprep /_noFileCopy_ 

_and add your user to shchema-administrators group._

