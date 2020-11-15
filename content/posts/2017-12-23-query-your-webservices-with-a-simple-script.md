---
title: Query your webservices with a simple python script
author: sitereal
type: post
date: 2017-12-23T17:22:13+00:00
url: /2017/12/23/query-your-webservices-with-a-simple-script/
featured_image: /wp-content/uploads/2017/12/web-services.png
pipdig_meta_geographic_location:
  - Madrid
categories:
  - Automation
  - Linux
  - Programming
  - Python
  - Site reliability

---
Hello all!!

I usually need yo check webservices. I check the http code and the time it takes to give me back the result of the soap query. It works only with ssl.

To execute the code:

python ./query_webservice.py -file /tmp/file.xml -host ws.example.com -context /context/ws -soapv v1.1

You will need to have a valid file.xml and specify the soap version

The output result:

HTTP_CODE: 200 HEALTH: OK  
Exec\_Total\_Time: 35 ms

<pre class="brush: python; title: ; notranslate" title="">#!/usr/bin/env python
#-*- coding: utf-8 -*-
# Koldo Oteo - (koteo [at] sitereliabilityengineer.io)
# December 18th 2017
import sys, time
import argparse
import httplib
import xml.dom.minidom

### Parse arguments
parser = argparse.ArgumentParser(description='Example:  ./query_webservice.py -file /tmp/file.xml \
                                 -host hostname.domain -context /context -soapv v1.1')
parser.add_argument('-file', action='store', dest='xml',
                    help='xml File Name')
parser.add_argument('-host', action='store', dest='host',
                    help='Webservice host')
parser.add_argument('-context', action='store', dest='context',
                    help='Webservice context')
parser.add_argument('-soapv', action='store', dest='soapv',
                    help='Soap Version v1.1 or v1.2')
# Print Parser Help
if len(sys.argv) == 1:
    parser.print_help()
    sys.exit(1)
param = parser.parse_args()
###

### FUNCTION TO Read xml File
def read_xml():
   with open(param.xml, 'r') as f:
      xmlmsg = f.read()
      return xmlmsg

###

### FUNCTION TO POST XML TO WEBSERVICE
def post_xml(xmlmsg):
   """HTTP XML Post request"""
   if param.soapv == "v1.2":
      headers = {"Content-type": "application/soap+xml","Content-Length": "%d" % len(xmlmsg), "charset": "utf-8", "SOAPAction": "", "User-Agent": "PythonSOAPClient"}
   elif param.soapv == "v1.1":
      headers = {"Content-type": "text/xml","Content-Length": "%d" % len(xmlmsg), "charset": "utf-8", "SOAPAction": "", "User-Agent": "PythonSOAPClient"}
   conn = httplib.HTTPSConnection(param.host)
   conn.request("POST", param.context, "", headers)
   # Send xml
   conn.send(xmlmsg)
   response = conn.getresponse()
   print "HTTP_CODE: %s  HEALTH: %s" % (response.status, response.reason)
   data = response.read()
   #resultxml = xml.dom.minidom.parseString(data)
   #print (resultxml.toprettyxml())
   conn.close()

###
# READ XML FILE
xmlmsg = read_xml()

# GET EXECUTION TOTAL TIME AND POST XML
start_time = time.time()
post_xml(xmlmsg)
print("Exec_Total_Time: %s ms" % int(round((time.time() - start_time) * 1000)))


###
</pre>