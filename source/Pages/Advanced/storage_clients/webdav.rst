.. _webdav:

****************
*webdav* clients
****************

This page describes how to use the webdav protocol. For an overview of storage clients, see :ref:`storage-clients`.

.. contents:: 
    :depth: 4
 

About WebDAV
============

The WebDAV protocol has the following advantages:

* It supports username & password authentication
* It uses the common port 443. Some overly strict firewalls may block outgoing traffic, but port 443 is so common that it is seldom blocked. However, using WebDAV to bypass firewalls should be seen as a temporary solution; it would be better to open up your institute's firewall to allow access to the dCache subnet.
* It can transfer data over a secure channel; other protocols like GridFTP do authentication over a secure channel but transfer data unencrypted.

It also has disadvantages:

* It is not a high performance transfer protocol. If this is important, use GridFTP instead.
* Support by WebDAV clients varies widely. Some operations (like renaming a file) may not work in certain clients and circumstances. Modifying/overwriting a file, although the WebDAV protocol supports it, doesn't work on dCache; you'll need to delete the old file and upload the new file instead.


Available WebDAV doors
======================

dCache has the following WebDAV doors:

.. comment: The following is a trick to get non-breaking spaces. See https://stackoverflow.com/questions/11830242/non-breaking-space

.. |nbsp| unicode:: 0xA0 
   :trim:

+------------------------------------------+---------------------------+-----------------------------+---------------------+
| URL including port                       | Authentication method     | Redirection behaviour       | Overwrites          |
+==========================================+===========================+=============================+=====================+
| https://webdav.grid.surfsara.nl:443      | Username/password         | Redirects on read           | Not |nbsp| allowed  |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://webdav.grid.surfsara.nl:2880     | Username/password         | No redirects                | Allowed             |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://webdav.grid.surfsara.nl:2882     | User certificate or proxy | Redirects on read and write | Not |nbsp| allowed  |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://webdav.grid.surfsara.nl:2883     | User certificate or proxy | No redirects                | Allowed             |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://webdav-cert.grid.sara.nl:443     | User certificate or proxy | No redirects                | Not |nbsp| allowed  |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://ipv4.grid.surfsara.nl:443        | Username/password         | Redirects on read           | Not |nbsp| allowed  |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://ipv4.grid.surfsara.nl:2880       | Username/password         | No redirects                | Allowed             |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://ipv4.grid.surfsara.nl:2882       | User certificate or proxy | Redirects on read and write | Not |nbsp| allowed  |
+------------------------------------------+---------------------------+-----------------------------+---------------------+
| https://ipv4.grid.surfsara.nl:2883       | User certificate or proxy | No redirects                | Allowed             |
+------------------------------------------+---------------------------+-----------------------------+---------------------+


Choosing a WebDAV door
----------------------

The most important consideration is whether you want to authenticate with username/password or with ``x509`` (certificate/proxy). Another important consideration is whether a WebDAV door should redirect or not.

Advantages of redirects:

* It's a form of load balancing, which improves the speed.
* Redirecting WebDAV doors do the authentication over HTTPS, but they redirect your client to an HTTP port. So the data transfer is unencrypted. This improves speed.

Disadvantages of redirects:

* File transfers are sent over HTTP, so they are not encryted. A "man in the middle" might be able to read the data, or even modify it in transit. If privacy is a concern, choose a door that does not redirect.
* Some WebDAV clients don't handle redirects very well.

Another consideration is whether you're using the door for parallel access.

``webdav.grid.surfsara.nl`` is a DNS round robin that will direct you to a (more or less) random host in a pool of WebDAV servers. So it is very well suited for parallel access.

Use ``webdav-cert.grid.sara.nl`` when you want to authenticate with a user certificate or proxy, and your institute's firewall blocks outgoing connections to port ``2882`` and ``2883``. It's a single virtual machine; don't use it for parallel processing.

Use ``ipv4.grid.surfsara.nl`` for storage clients that have problems with IPv6. It's a single virtual machine; don't use it for parallel processing.


Clients
=======

We've tested these WebDAV clients successfully with dCache:

* web browsers (read only)
* curl
* wget (read only)
* rclone (username/password only)
* cyberduck (GUI)

We'll describe how to use them below.


Web browsers
============

The easiest way to access dCache is with a normal web browser. You can point a browser like Firefox to https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/ or any of the other WebDAV doors listed in the table above. When the browser asks for a username and password, you can provide your Grid :abbr:`UI (User Interface)` (or :abbr:`CUA (SURFsara's Central User Administration)`) username and password. When you click on a listed file, it will be downloaded, if you're authorized to do so.

You can't upload to dCache with a normal web browser.


Curl & wget
===========

.. note:: To run the examples below you need to have a :abbr:`UI (User Interface)` (or :abbr:`CUA (SURFsara's Central User Administration)`) account that is configured within dCache and authorized to the data you want to access. Contact us if you need assistance with that.


Creating directories
--------------------

To create a directory with curl:

.. code-block:: console

   $curl --capath /etc/grid-security/certificates/ --fail --user homer \
        --request MKCOL https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/homer/directory

If on your system there are no Grid :abbr:`CA (Certificate Authority)` certificates available in ``/etc/grid-security/certificates/``, please read :ref:`host_certificates <host-certificates>`.


Transferring data
-----------------


---------
Uploading
---------

To copy a file from your local machine to dCache:

.. code-block:: console

   $curl --capath /etc/grid-security/certificates/ --fail --location --user homer \
        --upload-file zap.tar \
        https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/homer/
   $# replace homer with your username, lsgrid with your VO and zap.tar with your local file

The command will ask for the password of 'homer' on the command line. If you don't want to type the password each time, specify ``--netrc`` and store the password in the ``.netrc`` file in your home dir. Make sure it is not readable by others (``chmod 600 .netrc``). See ``man curl`` for more details. An example ``.netrc`` file is:

.. code-block:: console

   $ cat .netrc
   machine webdav.grid.surfsara.nl
   login <your_username>
   password <your_pwd>


.. note:: It is possible to specify the password on the command line like this: ``--user homer:password``. However, for security reasons this should be avoided on shared systems (like the :abbr:`UI (User Interface)`) because it allows other local users to read the password with the ``ps`` command.


-----------
Downloading
-----------

To copy a file from dCache to your local machine:

.. code-block:: console
  
   $curl --capath /etc/grid-security/certificates/ --fail --location --user homer \
        https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/homer/zap.tar \
        --output zap.tar
  
Or with ``wget``:
  
.. code-block:: console

   $wget --user=homer --ask-password --ca-directory=/etc/grid-security/certificates \
        https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/homer/zap.tar 

The ``--ca-directory`` is probably not necessary anymore. If it is, but you don't have an ``/etc/grid-security/certificates`` directory, you could specify ``--no-check-certificate``, but we don't recommend this.


-------------------------------------
Downloading with proxy authentication
-------------------------------------

To download a file while using a proxy to authenticate, you first have to create your proxy, see :ref:`startgridsession`.

Then use a command like this:

.. code-block:: console

   $curl --location --capath /etc/grid-security/certificates/ \
        --cert $X509_USER_PROXY --cacert $X509_USER_PROXY \
        https://webdav.grid.surfsara.nl:2882/pnfs/grid.sara.nl/data/lsgrid/homer/zap.tar

.. note:: It is possible that your proxy :abbr:`DN (Distinguished Name)` is mapped to another user account than your own :abbr:`CUA (SURFsara's Central User Administration)` user account. If you have permission issues with either username or proxy and not the other, contact us to check the user mapping.

Note: ``wget`` does not support certificate/proxy authentication.


-----------------
Partial downloads
-----------------

With Curl you can download only part of a file by specifying `--range`. Example:

.. code-block:: console

   $curl --fail --location --capath /etc/grid-security/certificates/ \
        --user homer \
	https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/homer/myfile \
	--range 0-4 \
	--output first-5-bytes 


Renaming
--------


-------------------------
With proxy authentication
-------------------------

.. code-block:: console

   $curl --capath /etc/grid-security/certificates/  --fail --location \
        --cert $X509_USER_PROXY --cacert $X509_USER_PROXY \
        --request MOVE \
        https://webdav.grid.surfsara.nl:2882/pnfs/grid.sara.nl/data/lsgrid/homer/oldfile \
        --header "Destination:https://webdav.grid.surfsara.nl:2882/pnfs/grid.sara.nl/data/lsgrid/homer/newfile"

File properties and locality are not changed. A file that is stored on tape (nearline) will stay on tape, even if it is moved to a directory for disk-only files.


-------------------------------------
With username/password authentication
-------------------------------------

.. code-block:: console

   $curl --capath /etc/grid-security/certificates/  --fail --location-trusted \
        --user homer \
        --request MOVE \
        https://webdav.grid.surfsara.nl:2880/pnfs/grid.sara.nl/data/lsgrid/homer/oldfile \
        --header "Destination:https://webdav.grid.surfsara.nl:2880/pnfs/grid.sara.nl/data/lsgrid/homer/newfile"

Please note the differences with the previous example:

* ``--location-trusted`` will send the username and password also to the destination server.
* Port ``2880`` is used for username/password authentication.


Removing data
-------------

Deleting a file from dCache:

.. code-block:: console

   $curl --capath /etc/grid-security/certificates/ --user homer --location \
        --request DELETE https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/homer/zap.tar 


Querying file properties
------------------------

With curl and a dCache WebDAV door, it's possible to request file properties. This works both with username/password and proxy authentication, provided you use the correct port (``443`` or ``2880`` for username/password, ``2882`` or ``2883`` for proxy authentication). 


--------
Locality
--------

This example shows how to query the file locality: whether a file is online or nearline (on tape). This example uses username/password authentication:

.. code-block:: console

   $echo -e '<?xml version="1.0"?>\n
            <a:propfind xmlns:a="DAV:">
            <a:prop><srm:FileLocality xmlns:srm="http://srm.lbl.gov/StorageResourceManager"/></a:prop>
            </a:propfind>' \
   | curl --silent --fail --capath /etc/grid-security/certificates/ \
          --user homer --request PROPFIND \
          https://webdav.grid.surfsara.nl:2880/pnfs/grid.sara.nl/data/lsgrid/homer/zap.tar \
          --header "Content-Type: text/xml" --upload - \
   | xmllint -format -

See :ref:`staging` for more information about file locality.


-----------------
Adler32 checksums
-----------------

This example shows how to get the checksum of a stored file. dCache uses Adler32 checksums by default. 

The returned checksum comes from the dCache database, so it is a very efficient way to check your files. dCache does checksum checks on most operations, so you can safely assume the checksum matches the stored file.

.. code-block:: console

   $curl --head --header 'Want-Digest: ADLER32' --silent --fail --capath /etc/grid-security/certificates/ \
        --user homer \
        https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/home/myfile \
   | grep 'adler32='

Here an example output:

.. code-block:: console

   Digest: adler32=46fd067a


Here is an alternative way to query an Adler32 checksum:

.. code-block:: console

   $echo -e '<?xml version="1.0"?>\n
            <a:propfind xmlns:a="DAV:">
            <a:prop><srm:Checksums xmlns:srm="http://www.dcache.org/2013/webdav"/></a:prop>
            </a:propfind>' \
   | curl --silent --fail --capath /etc/grid-security/certificates/ \
          --user homer --request PROPFIND \
          https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/homer/myfile \
          --header "Content-Type: text/xml" --upload - \
   | xmllint -format - \
   | egrep -o '<ns1:Checksums>.*</ns1:Checksums>'

Here is an example of the expected output:

.. code-block:: console

   $<ns1:Checksums>adler32=46fd067a</ns1:Checksums>


-------------
MD5 checksums
-------------

The dCache grid storage at SURFsara is configured to use only Adler32 checksums. Some other storage services may use MD5 checksums. This complicates things a bit because they are base64 encoded, as prescribed by RFC 3230.

.. code-block:: console

   $curl --head --header 'Want-Digest: MD5' --silent --fail --capath /etc/grid-security/certificates/ \
        --user homer \
        https://pn1.cdi.surfsara.nl:2880/cdi/users/homer/myfile \
   | grep -o 'md5=.*' \
   | sed -e 's/md5=//' -e 's/[\r\n]*$//' \
   | base64 --decode \
   | xxd -p

The output should look similar to this:

.. code-block:: console

   0f43fa5a262c476393018f7329080fa7

An alternative way to query an MD5 checksum:

.. code-block:: console

   $echo -e '<?xml version="1.0"?>\n
            <a:propfind xmlns:a="DAV:">
            <a:prop><srm:Checksums xmlns:srm="http://www.dcache.org/2013/webdav"/></a:prop>
            </a:propfind>' \
   | curl --silent --fail --capath /etc/grid-security/certificates/ \
          --user homer --request PROPFIND \
          https://pn1.cdi.surfsara.nl:2880/cdi/users/homer/myfile \
          --header "Content-Type: text/xml" --upload - \
   | xmllint -format - \
   | egrep -o '<ns1:Checksums>md5=.*</ns1:Checksums>' \
   | sed -e 's#<ns1:Checksums>[^=]*=\([^<]*\)</ns1:Checksums>#\1#' \
   | base64 --decode \
   | xxd -p

Queries can be combined to reduce transaction overhead:

.. code-block:: console

   $echo -e '<?xml version="1.0"?>\n
            <a:propfind xmlns:a="DAV:">
            <a:prop><srm:RetentionPolicy xmlns:srm="http://srm.lbl.gov/StorageResourceManager"/></a:prop>
            <a:prop><srm:AccessLatency xmlns:srm="http://srm.lbl.gov/StorageResourceManager"/></a:prop>
            <a:prop><srm:FileLocality xmlns:srm="http://srm.lbl.gov/StorageResourceManager"/></a:prop>
            <a:prop><srm:Checksums xmlns:srm="http://www.dcache.org/2013/webdav"/></a:prop>
            </a:propfind>' \
   | curl ...


Rclone
======

Rclone is a command line tool that you can download from https://rclone.org/downloads/. It works on many platforms and it can talk to many storage systems besides WebDAV.

Advantages of Rclone are:

* It can sync directories, like rsync does
* It uses parallel transfers, 4 by default, to get a better performance

There is also a disadvantage: it uses only username/password authentication; not X509 certificate/proxy authentication. You'll have to use your CUA credentials and write to a directory where you are permitted to write.

Because of this, Rclone is best suited for uploading or downloading large datasets; lacking X509 client authentication, it's not suited for batch processing.

The first time you use rclone, you need to make a profile with ``rclone config``.

You can use for example ``https://webdav.grid.surfsara.nl:443/pnfs/grid.sara.nl/data/lsgrid/homer`` (for performance) or ``https://webdav.grid.surfsara.nl:2880/pnfs/grid.sara.nl/data/lsgrid/homer`` (with encrypted transport).

An example of a profile:

.. code-block:: console

   [dcache]
   url = https://webdav.grid.surfsara.nl:2880/pnfs/grid.sara.nl/data/lsgrid/homer
   vendor = other
   user = homer
   pass = *** ENCRYPTED ***

An example of using rclone to copy a directory:

.. code-block:: console

   $rclone copy mydir dcache:rclone-test

More information on how to use ``rclone`` with WebDAV is here: https://rclone.org/webdav/. There are also graphical user interfaces to ``rclone``; one is `RcloneBrowser <https://github.com/mmozeiko/RcloneBrowser>`_.


Graphical access with Cyberduck
===============================
  
To work with WebDAV on Windows or Mac OS X, you can install **Cyberduck** from here: https://cyberduck.io/. Please note that the App store package costs money; the download from the website is free, but will ask for a donation.

* Download the .zip file, open it, and drag the .app file into your Applications folder to install it. 
* Open a WebDAV (HTTP/SSL) connection and connect to the server with your :abbr:`UI (User Interface)` account username and password:

  .. code-block:: bash

     https://webdav.grid.surfsara.nl/pnfs/grid.sara.nl/data/lsgrid/ # replace lsgrid with your VO

.. image:: /Images/cyberduck.png
	:align: center

Cyberduck with a user certificate
---------------------------------

Normally, one would authenticate to dCache using a user certificate or proxy. dCache determines your identity based either on your user certificate or proxy DN, or on your VOMS credentials. However, if you authenticate with your CUA username & password, that identity might not be the same and you may not have access to your own data.

To work around this, it may be useful to have Cyberduck authenticate you using your user certificate. 

.. note:: Most users are authenticated based on the VOMS credentials of their proxy. Since you will not use a *VOMS* proxy but a certificate, this identity mapping won't work and you may not have access. Instead, we may need to map your *DN* onto the desired identity instead of your VOMS credentials. If you want to use Cyberduck with certificate authentication, contact us so that we can map your DN to the desired identity.

Here is how to configure Cyberduck for certificate authentication on OS X:

First, import your user certificate in p12 format into the Keychain Access. It should look something like this:

.. image:: /Images/cyberduck-usercert-1.png
	:align: center

Second, go to Cyberduck and create a bookmark with these settings:

.. image:: /Images/cyberduck-usercert-2.png
	:align: center

If your institute blocks outgoing traffic to port ``2882`` and ``2883``, you can use server ``webdav-cert.grid.sara.nl`` and port ``443``, as described at the top of this page.

Right-click the bookmark and choose "Connect to server".

.. image:: /Images/cyberduck-usercert-3.png
	:align: center

Choose your certificate.

.. image:: /Images/cyberduck-usercert-4.png
	:align: center
	:scale: 50 %


Mounting WebDAV with Mountain duck
----------------------------------

Mountain Duck is a WebDAV client that can mount WebDAV in OS-X and Windows. We've had some mixed results using it with dCache, so you should test it before trusting it with your data. You can find it at https://mountainduck.io/.
