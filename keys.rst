.. _keys:

How To Set Up SSH Keys and SSH-config file
==================================

SSH keys provide a more secure way of logging into a virtual private server with  SSH  than using a  password alone.
While a  password can eventually be cracked with a brute force attack, SSH keys are nearly impossible to decipher by brute force alone.
Generating a key pair provides you with two long string of characters: a public and a private key. You can place the public key on any server, and then unlock it by connecting to it with a client that already has the private key.
When the two matches up, the system unlocks without the need for a password. You can increase security even more by protecting the private key with a passphrase.

1. Create the RSA Key Pair
----------------- 

The first step is to create the key pair on the client machine (there is a good chance that this is your computer) with ``ssh-keygen -t rsa``.


2. Store the Keys and Passphrase  
-----------------

Once you have entered the generate key command, you get a few more questions:

::

  Enter file in which to save the key ($HOME/.ssh/hostname_id_rsa):


You can press enter here, saving the file to the user .ssh directory

::

  Enter passphrase (empty for no passphrase):


It’s up to you whether you want to use a passphrase. 
Entering a passphrase does have its benefits: the security of a key, no matter how encrypted, still depends on the fact that it is not visible to anyone else. 
Should a passphrase protected private key fall into an unauthorized user’s possession, they will be unable to log in to its associated accounts until they figure out the passphrase, buying the hacked user some extra time.
The entire key generation process looks like this:

::

  ssh-keygen -t rsa
  Generating public/private rsa key pair. Enter file in which to save the key ($HOME/.ssh/hostname_id_rsa):
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in $HOME/.ssh/hostname_id_rsa. 
  Your public key has been saved in $HOME/.ssh/hostname_id_rsa.pub.

The public key is now located in ``$HOME/.ssh/hostname_id_rsa.pub``. The private key (identification) is now located in ``$HOME/.ssh/hostname_id_rsa``.

3. Copy the Public Key to the virtual server
-----------------

Once you generated the key pair, it’s time to place the public key on the virtual server that we want to use.
Make sure to replace the example username and IP address below. You can directly paste the keys in the virtual server using SSH:

::

  cat ~/.ssh/hostname_id_rsa.pub | ssh user@12.34.56.78 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
  The authenticity of host’ 12.34.56.78 (12.34.56.78)’ can’t be established.
  RSA key fingerprint is b1:2d:33:67:ce:35:4d:5f:f3:a8:cd:c0:c4:48:86:12.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added ‘12.34.56.78’ (RSA) to the list of known hosts. 
  user@12.34.56.78’s password: 

Now try logging into the machine, with ``ssh user@12.34.56.78``, and check within ``~/.ssh/authorized_keys`` to make sure we haven’t added extra keys that you weren’t expecting.

Add key to ssh-agent permanently (between reboots only for Mac):

::

  ssh-add -k ~/.ssh/hostname_id_rsa

check if the key was added:

::

  ssh-add –l

Now you can go ahead and log into ``user@12.34.56.78``, and you will not be
prompted for a password,

::

  ssh –XY user@12.34.56.78
  
Here, we are enabling trusted X11 forwarding by using ``ssh -XY``.

Uses for ``~/.ssh/config``
-----------------

If you frequently deal with sessions on multiple machines, SSH ends up being one of the most often used tools.
To abbreviate a long server name include this in your ``~/.ssh/config`` file:

::

  Host server1
  User $SERVER1_USERNAME
  HostName server1.23.45.67.89
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/server1_id_rsa 
  
  Host server2
  User $SERVER2_USERNAME
  HostName server2.34.56.78.90
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/server2_id_rsa

The above allows you to type:

::

  ssh –XY server1

And no password is required. If you have and SSH Server that’s only accessible to you via an SSH session on an intermediate machine, which is the prevailing situation with remote private networks, you can automate that in ``~/.ssh/config`` too. 
Say you can’t reach ``server2.34.56.78.90`` directly, but you can reach some other SSH server on the same private subnet that is publically accessible, say ``server1.23.45.67.89``.
Then, you can add the following lines to *proxyjump* to ``server2`` using ``server1`` server:

::
  Host jumpserver2
  User $SERVER2_USERNAME
  Hostname server2.34.56.78.90
  UseKeychain yes
  AddKeysToAgent yes
  ProxyCommand ssh -X -i ~/.ssh/server1_id_rsa server1.23.45.67.89.edu -W %h:%p
  IdentitiesOnly yes

This allows you to just type:

::

  ssh –XY jumpserver2

Finally, your ``~/.ssh/config`` file may look like this:

::

  Host server1
  User $SERVER1_USERNAME
  HostName server1.23.45.67.89.edu
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/server1_id_rsa 

  Host jumpserver2
  User $SERVER2_USERNAME
  Hostname server2.34.56.78.90.edu
  UseKeychain yes
  AddKeysToAgent yes
  ProxyCommand ssh -X -i ~/.ssh/server1_id_rsa server1.23.45.67.89.edu -W %h:%p
  IdentitiesOnly yes

  Host server2
  User $SERVER2_USERNAME
  HostName server2.34.56.78.90.edu 
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/server2_id_rsa
