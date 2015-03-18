---
layout: post
title:  "GPG keys management"
excerpt_separator: <!--excerpt-->
date:   2014-09-30
categories: articles
tags:
- email
- security
---

Manage your GPG keys to encrypt or sign documents.
<!--excerpt-->

# Why using GPG?

There is no doubt that nowadays, to protect your privacy, you can not trust anyone. Neither companies, nor governements, ... etc. You have to act by yourself. (I let you re-read the Snowden/Manning/*whistleblowers stories and conclusions)

Fortunatly, there are great tools for that.
New tools ? NO ! They are around for years and years!

Among them, there is the standard [GnuPG](https://www.gnupg.org/) (GPG) binaries which are the GNU port of the OpenPGP suite that is itself an open port of the [PGP](http://fr.wikipedia.org/wiki/Pretty_Good_Privacy) suite, initially written in 1991 by [Phil Zimmermann](http://philzimmermann.com/EN/background/index.html).

Now, let's create and publish a key that will help you to sign/encrypt your communication (i.e via your mail user agent like [Gnus](http://qsdfgh.com/articles/gnus-configuration-example.html)) or your files.

# Create a GnuPG key

## Generate your key

In order to generate a new GPG key, the command `gpg --gen-key` will help you by asking a few questions.

Here is a sample session:

{% highlight sh %}
$ gpg --gen-key
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 2048
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: My Name
Email address: me@mail.com
Comment:
You selected this USER-ID:
    "My Name <me@mail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
pub   2048R/3732BE06 2014-09-30
      Key fingerprint = 94B2 5EAC A604 00C0 44B4  2CBB D139 51B8 3732 BE06
uid                  My Name <me@mail.com>
sub   2048R/CA23BF1E 2014-09-30
{% endhighlight %}

Now you have a new key, and its ID is `3732BE06`.

In order to check the changes, run `gpg --list-keys`:

{% highlight sh %}
$ gpg --list-keys
/home/me/.gnupg/pubring.gpg
-----------------------------
pub   2048R/3732BE06 2014-09-30
uid                  My Name <me@mail.com>
sub   2048R/CA23BF1E 2014-09-30
{% endhighlight %}

## Add other identities to your key

Sometimes, you want to sign your communications or encrypt your file with a specific identity.
Let's say that you have your own company and beside your personnal identity (*My Name <me@mail.com>*), you want to use your professional identity (*My Name <me@company-mail.com>*).

Two solutions:

- You create another standalone key by starting a new `gpg --gen-key` session
- You add a new identity to your existing key

In order to achieve the second solution, you just have to edit your key like this:

{% highlight sh %}
$ gpg --edit-key 3732BE06
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1). My Name <me@mail.com>

gpg> adduid
Real name: My Name
Email address: me@company-name.com
Comment: Company
You selected this USER-ID:
    "My Name (Company) <me@company-name.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

You need a passphrase to unlock the secret key for
user: "My Name <me@mail.com>"
2048-bit RSA key, ID 3732BE06, created 2014-09-30


pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1)  My Name <me@mail.com>
[ unknown] (2). My Name (Company) <me@company-name.com>

gpg> save
{% endhighlight %}

Again, check the changes with `gpg --list-keys`:

{% highlight sh %}
$ gpg --list-keys
/home/me/.gnupg/pubring.gpg
-----------------------------
pub   2048R/3732BE06 2014-09-30
uid                  My Name (Company) <me@company-name.com>
uid                  My Name <me@mail.com>
sub   2048R/CA23BF1E 2014-09-30
{% endhighlight %}

## Choose the main identity

Now that you have your "multiple identities" key, you may define a main identity.

{% highlight sh %}
$ gpg --edit-key 3732BE06
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1). My Name (Company) <me@company-name.com>
[ultimate] (2)  My Name <me@mail.com>

gpg> uid 2

pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1). My Name (Company) <me@company-name.com>
[ultimate] (2)* My Name <me@mail.com>

gpg> primary

You need a passphrase to unlock the secret key for
user: "My Name (Company) <me@company-name.com>"
2048-bit RSA key, ID 3732BE06, created 2014-09-30


pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1)  My Name (Company) <me@company-name.com>
[ultimate] (2)* My Name <me@mail.com>

gpg> save
{% endhighlight %}

Check the changes with `gpg --list-keys` and note that the main identity is now on top:

{% highlight sh %}
$ gpg --list-keys
/home/me/.gnupg/pubring.gpg
-----------------------------
pub   2048R/3732BE06 2014-09-30
uid                  My Name <me@mail.com>
uid                  My Name (Company) <me@company-name.com>
sub   2048R/CA23BF1E 2014-09-30
{% endhighlight %}

## Set your keys' trust level

The next step is to define a trust level for your key:

{% highlight sh %}
$ gpg --edit-key 3732BE06
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1). My Name <me@mail.com>
[ultimate] (2)  My Name (Company) <me@company-name.com>

gpg> uid 1

pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1)* My Name <me@mail.com>
[ultimate] (2)  My Name (Company) <me@company-name.com>

gpg> trust
pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1)* My Name <me@mail.com>
[ultimate] (2)  My Name (Company) <me@company-name.com>

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y

pub  2048R/3732BE06  created: 2014-09-30  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/CA23BF1E  created: 2014-09-30  expires: never       usage: E
[ultimate] (1)* My Name <me@mail.com>
[ultimate] (2)  My Name (Company) <me@company-name.com>

gpg> save
{% endhighlight %}

And repeat the same operation for all your different identities.

# Send your key to main keyservers

Once you are done with the creation of your key and its setup, you have to publish them on signature servers.
It is necessary (mandatory) so that the different clients that will check your key can retrieve it and ensure the validity of the encrypted document (communication or files).

You can publish it on several trusted server. To do so, just run

{% highlight sh %}
gpg --send-key 3732BE06
{% endhighlight %}

This will take the main key server defined in your `~/.gnupg/gpg.conf` (you can modify it if you want).
But maybe you want to publish it occasionnaly on a specific serrver. For this, just add the `--keyserver` option:

{% highlight sh %}
gpg --keyserver hkp://pgp.mit.edu --send-key 3732BE06
{% endhighlight %}

Now you are ready to sign/encrypt whatever you want and send it to whoever you want (well ... not everybody, of course).

# Export and import keys

Let's say that you have two computer on which you want to be able to sign/encrypt some data with the same keyring.

For this, you can export your keys from the first one and import them in the second one.

To export it, simply run:

{% highlight sh %}
gpg --export -a 3732BE06 > mykey.pub
gpg --export-secret-key -a 3732BE06 > mykey.priv
{% endhighlight %}

Now, to import them ion the other side, run:

{% highlight sh %}
gpg --import mykey.pub
gpg --allow-secret-key-import --import mykey.priv
{% endhighlight %}


# Now, what can I do with my key?

Here are some practical examples of using your GPG key.

## Configure your MUA to sign your emails

Most of the decent MUA (mail user agents, or mail reader if you prefer) support encryption.
I will not detail the configuration for each of them, but here are some useful links for each MUA:


- [Gnus](http://qsdfgh.com/articles/gnus-configuration-example.html#sec-2-2-6)
- [Mutt](http://www.linuxjournal.com/content/encrypt-your-dog-mutt-and-gpg)
- Thunderbird, with the [Enigmail](https://www.enigmail.net/home/index.php) plugin
- and so on ... (I think Google can find informations for your MUA)

## Encrypt and decrypt a file

GPG allows you to encrypt and decrypt some files in order to protect them with a password.
It can be very useful if you want to store some sensible informations and/or credentials.

To encrypt a file using your key, just run:

{% highlight sh %}
gpg --output secretfile.txt.gpg --encrypt --recipient myfriend@mail.com secretfile.txt
{% endhighlight %}

If you do not wat to use your key, but use a standalone passphrase (symmetric encryption), use:

{% highlight sh %}
gpg --output secretfile.txt.gpg --symmetric secretfile.txt
{% endhighlight %}

And to decrypt your file, use:

{% highlight sh %}
gpg --output secretfile.txt --decrypt secretfile.txt.gpg
{% endhighlight %}

Easy, isn't it ?

# Revoke

Let's say that some day you change your email address and you created a new key.
The previous one is not used anymore and it disturbs you to have an unused key still alive (and you are right).

To delete it from your keyring and from the network, here are some simple steps to follow.

First, generate a revocation key:

{% highlight sh %}
$ gpg --gen-revoke 3732BE06 > revoke.txt

sec  2048R/3732BE06 2014-09-30 My Name <me@mail.com>

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 3
Enter an optional description; end it with an empty line:
>
Reason for revocation: Key is no longer used
(No description given)
Is this okay? (y/N) y

You need a passphrase to unlock the secret key for
user: "My Name <me@mail.com>"
2048-bit RSA key, ID 3732BE06, created 2014-09-30

ASCII armored output forced.
Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!
{% endhighlight %}

Then import the revocation key:

{% highlight sh %}
$ gpg --import revoke.txt
gpg: key 3732BE06: "My Name <me@mail.com>" revocation certificate imported
gpg: Total number processed: 1
gpg:    new key revocations: 1
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
{% endhighlight %}

And finally publish the changes by sending it to the keys servers you published it:

{% highlight sh %}
gpg --send-key 3732BE06
{% endhighlight %}

(Don't forget other servers you reached with the `--keyserver` earlier)

The last step is to delete the key from your local keyring:

{% highlight sh %}
$ gpg --delete-secret-key 3732BE06
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  2048R/3732BE06 2014-09-30 My Name <me@mail.com>

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y
{% endhighlight %}

{% highlight sh %}
$ gpg --delete-key 3732BE06
gpg (GnuPG) 1.4.12; Copyright (C) 2012 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  2048R/3732BE06 2014-09-30 My Name <me@mail.com>

Delete this key from the keyring? (y/N) y
{% endhighlight %}
