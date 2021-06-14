---
layout: post
text-align: justify
title:  "SMB Config Change"
date:   2021-03-15
---

Subcategory: **[Changes & Installations: Getting Kali ready for OSCP]({% post_url 2021-03-01-Changes & Installations: Getting Kali ready for OSCP %})**


These are instructions for Kali version 2020.1 and 2020.3. 

The above versions of Kali require a configuration to the smbclient or connection fails with the following error.

![My helpful screenshot](/assets/smbchange.png) 

# The Fix 

- Go to the file **smb.conf** 

<pre><code class="console">
sudo gedit /etc/samba/smb.conf

</code></pre>

- Edit the file by adding the following settings under “GLOBAL”: <br/>
<pre><code class="console">
client min protocol = CORE 
client max protocol = SMB3

</code></pre>

