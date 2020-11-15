---
title: 'Automation: P12 keystore generation'
author: sitereal
type: post
date: 2017-12-10T19:23:21+00:00
url: /2017/12/10/automation-p12-keystore-generation/
featured_image: /wp-content/uploads/2017/12/openssl1.png
categories:
  - Automation
  - Python

---
## Some time ago, I create a script to generate a p12 automatically.

Basically, you could create a crontab entry executing the script and it will send a mail witht the p12 store.

At this moment, the script only works with unauthenticated smtp´s. In the future will also create a java key store.

## <a id="user-content-steps" class="anchor" href="https://github.com/SiteReliabilityEngineering/sre/blob/master/auto_p12_jks/README.md#steps" aria-hidden="true"></a>Steps:

  1. You need to create a .zip with a private key .key file into the zip.
  2. .crt file with the certificate
  3. A params.txt file with 3 lines:  
    In the first Line you have to put the password of the pem file with the private key.  
    In the second Line you have to put the password of the privatekey and p12.  
    In the third Line, an alias for the privatekey.  
    In the fourth Line, you have to put the mail where is going to be sent the .p12 file.  
    Example:  
    PemPassword  
    p12\_and\_privatekey_password  
    alias  
    <example@email.com>

## <a id="user-content-you-wil-also-need-to-configure-some-vars-inside-the-py-file" class="anchor" href="https://github.com/SiteReliabilityEngineering/sre/blob/master/auto_p12_jks/README.md#you-wil-also-need-to-configure-some-vars-inside-the-py-file" aria-hidden="true"></a>You wil also need to configure some vars inside the .py File:

<pre><div class="codecolorer-container text blackboard" style="overflow:auto;white-space:nowrap;width:650px;">
  <table cellspacing="0" cellpadding="0">
    <tr>
      <td class="line-numbers">
        <div>
          1<br />2<br />3<br />4<br />
        </div>
      </td>
      
      <td>
        <div class="text codecolorer">
          autop12_dir is the directory where you are going to upload the .zip files. <br />
          smtp_host is the host of the smtp server, <br />
          smtp_port with the smtp port. <br />
          mail_from where you put the e-mail address of the sender.
        </div>
      </td>
    </tr>
  </table>
</div>

</pre>

&nbsp;

Here is the code:

[https://github.com/SiteReliabilityEngineering/sre/blob/master/auto\_p12\_jks][1]

&nbsp;

&nbsp;

<pre class="brush: python; title: ; notranslate" title="">#!/bin/env python
# -*- coding: utf-8 -*-
#  Koldo Oteo Orellana (koteo [at] sitereliabilityengineer.io)
#  Thanks to Eitan for the email attachment! code from https://stackoverflow.com/questions/25346001/add-excel-file-attachment-when-sending-python-email
#  09-June-2017
#
# .py program for automating the creation of .p12 files. You need to create a .zip with a private key .key file,
# a .crt file with the certificate, and a params.txt file with 3 lines:
# In the first line you put the password of the privatekey and p12, in the second an alias for the privatekey
# and in the third line, you put the mail where is going to be sent the .p12 file.
# You wil also need to configure some vars: autop12_dir is the directory where you are going to upload the
# .zip files. smtp_host is the host of the smtp server, and smtp_port with the smtp port. There's also mail_from
# where you put the e-mail address of the sender.
#
#
from OpenSSL.crypto import FILETYPE_PEM, load_certificate, PKCS12, load_privatekey, PKCS12Type
from zipfile import ZipFile
import os, smtplib, shutil
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email.utils import formatdate
from email import encoders


## VARS
autop12_dir = '/home/USER/AUTO_P12'
#  SMTP SERVER//PORT CONFIGURATION. SET PORT AS INT
smtp_host = 'smtp.host'
smtp_port = 25
mail_from = 'koteo &lt;koteo [at] sitereliabilityengineer.io&gt;'
mail_text = 'P12 attached to this e-mail message'
##


# send mail with attachment
def s_mail(mail_from, mail_to, smtp_host, smtp_port, certdir, p12file, mail_text):
     mail = MIMEMultipart()
     mail['From'] = mail_from
     mail['To'] = mail_to
     mail['Date'] = formatdate(localtime = True)
     mail['Subject'] = 'Requested p12'
     mail.attach(MIMEText(mail_text))

     part = MIMEBase('application', "octet-stream")
     part.set_payload(open(p12file, "rb").read())
     encoders.encode_base64(part)
     part.add_header('Content-Disposition', 'attachment; filename="{0}"'.format(os.path.basename(p12file)))
     mail.attach(part)

     mta = smtplib.SMTP(smtp_host, smtp_port)
     mta.sendmail(mail_from, mail_to, mail.as_string())
     mta.quit()

# Read params from file and  split the params into a list
def read_params(paramfile):
    try:
        with open(paramfile, 'r') as param:
            data = param.read()
            return data.split()
    except:
        print ("Couldn't open file: %s" % (paramfile))

# Return a list with  files with .zip extension
def get_zips():
    zipfiles= os.listdir(autop12_dir)
    return [ autop12_dir +'/' + filez for filez in zipfiles if filez.lower().endswith('zip') ]

# unzip file to directory. I do the rstrip to  create a directori getting rid of .zip
def unzip_files():
    for zipf in zipfiles:
        zip_ref = ZipFile(zipf, 'r')
        zip_ref.extractall(zipf.rstrip('.zip.ZIP'))
        zip_ref.close()

# We open the public key and serialize it as a Base64-encoded representation
def open_pub(pubfile):
    try:
        with open(pubfile, 'r') as pub:
            x509obj = pub.read()
            return load_certificate(FILETYPE_PEM, x509obj)
    except:
        print ("Couldn't open file: %s" % (pubfile))

# We open the private key and serialize it as base64-encoded representation
def open_privkey(keyfile):
    try:
        with open(keyfile, 'r') as privkey:
            x509obj = privkey.read()
            if x509obj.find('ENCRYPTED') &gt; -1:
                return load_privatekey(FILETYPE_PEM, x509obj, pem_pass)
            else:
                return load_privatekey(FILETYPE_PEM, x509obj)
    except:
        print ("Couldn' t open file: %s" % (keyfile))

# Create p12 File
def p12create(p12file):
    try:
        p12 = PKCS12()
        p12.set_certificate(pub)
        p12.set_privatekey(privkey)
        p12.set_friendlyname(alias)
        p12data = p12.export(passw, iter=2048, maciter=1)
    except:
        return 'Failed to create p12 Object of p12create() method'
    try:
        newfile = open(p12file, 'wb+')
        newfile.write(p12data)
        newfile.close()
    except:
        print ("Problem in p12create method. Couldn't write file: %s" % (p12file))

################################################################################
# Here we start executing the functions

# I get a list called zipfiles with the  full path of .zip files
zipfiles = get_zips()
unzip_files()

#
for element in zipfiles:
    certdir = element.rstrip('.zip.ZIP')
    certfiles = os.listdir(certdir)
    for file in certfiles:
# I put the content of the params.txt file into a List and then I convert
# every elemento to byte string' and saving into two vars
        if file == 'params.txt':
            pem_pass = read_params(certdir + '/' + 'params.txt')[0].encode('utf-8')
            passw = read_params(certdir + '/' + 'params.txt')[1].encode('utf-8')
            alias = read_params(certdir + '/' + 'params.txt')[2].encode('utf-8')
            mail_to = read_params(certdir + '/' + 'params.txt')[3].encode('utf-8')
# I create the x509 object of the certificate and private key
        elif file.endswith('.crt') or file.endswith('.CRT'):
            pub = open_pub(certdir + '/' + file)
        elif file.endswith('.key') or file.endswith('.KEY'):
            privkey = open_privkey(certdir + '/' + file)
        else:
            pass

# I create an object with the path and name of the .p12 file and after that I send the mail
    p12file = certdir + '/' + alias +'.p12'
    p12create(p12file)
    s_mail(mail_from, mail_to, smtp_host, smtp_port, certdir, p12file, mail_text)

# deleting zip files
    try:
        os.remove(element)
    except OSError as err:
        print("OS error: {0}".format(err))
# deleting directories
    try:
        shutil.rmtree(certdir)
    except OSError as err:
        print("OS error: {0}".format(err))

################################################################################
</pre>

 [1]: https://github.com/SiteReliabilityEngineering/sre/blob/master/auto_p12_jks