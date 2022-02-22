---
title: "checkcloudstack.py"
---

```toc

```

# Cloudstack Check with python
--------------------------------------------------------------------------------------------------------

#!/usr/bin/env python
# checkcloudstack.py 2

import sys
import urllib2
import urllib
import hashlib
import hmac
import base64
import json

def getReq (command):
  baseurl='http://localhost:8080/client/api?'
  request={}
  request['command']=command
  request['response']='json'
  request['apikey']='L9vB6WEuA4Lv07vQ_ITN6aWXSogEX4_3c2UUck9RlqK0p0bP4N92xBJMHSxeXOYSEaqeRPvhAX2vw0VOPs3Hug'
  secretkey='ubMFyOh_GBRGsD-UbMSH6BgdCh5_VOj-7qtsxzaxqQQZkcmBb_1Sy1l27AqYUkNBX_6oGwM8Vrm8yEVXkyAvJg'

  request_str='&'.join(['='.join([k,urllib.quote_plus(request[k])]) for k in request.keys()])

  sig_str='&'.join(['='.join([k.lower(),urllib.quote_plus(request[k].lower().replace('+','%20'))])for k in sorted(request.iterkeys())])

  sig=hmac.new(secretkey,sig_str,hashlib.sha1)
  sig=hmac.new(secretkey,sig_str,hashlib.sha1).digest()
  sig=base64.encodestring(hmac.new(secretkey,sig_str,hashlib.sha1).digest())
  sig=base64.encodestring(hmac.new(secretkey,sig_str,hashlib.sha1).digest()).strip()
  sig=urllib.quote_plus(base64.encodestring(hmac.new(secretkey,sig_str,hashlib.sha1).digest()).strip())

  req = baseurl + request_str + '&signature=' + sig

  return req


def getHealth_01 (keyword, command):
  print ('[{}]'.format(keyword))

  req = getReq (command)

  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  responseKeyword = "{}response".format(command.lower())

  # print (json.dumps(resultParsed[responseKeyword], indent=2, sort_keys=True))

  for obj in resultParsed[responseKeyword][keyword]:
    allocationstateExist = "allocationstate" in obj

    if allocationstateExist:
      print ("{}: {}".format (obj['name'], obj['allocationstate']))
    else:
      print ("{}: {}".format (obj['name'], obj['state']))
  print("")


def getHealth_02 (keyword, command):
  print ('[{}]'.format(keyword))

  req = getReq (command)
  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  responseKeyword = "{}response".format(command.lower())

  # print (json.dumps(resultParsed[responseKeyword], indent=2, sort_keys=True))

  for obj in resultParsed[responseKeyword][keyword]:
    hosttagsExist = "hosttags" in obj

    if hosttagsExist:
      print ("{}: {} / {}".format (obj['hosttags'], obj['state'], obj['resourcestate']))
    else:
      print ("{}: {} / {}".format (obj['name'], obj['state'], obj['resourcestate']))

  print("")


def getHealth_03 (keyword, command):
  print ('[{}]'.format(keyword))
  print ("if VM is not Running, then print!")
  print('--------------------------------------')

  req = getReq (command)
  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  responseKeyword = "{}response".format(command.lower())

  # print (json.dumps(resultParsed[responseKeyword], indent=2, sort_keys=True))

  for obj in resultParsed[responseKeyword][keyword]:
    if (obj['state'] != 'Running'):
      print ("{}: {}".format (obj['name'], obj['state']))

  print("")


def getHealth_04 (keyword, command):
  print ('[{}]'.format(keyword))
  print ("if level is not INFO, then print!")
  print('--------------------------------------')

  req = getReq (command)
  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  responseKeyword = "{}response".format(command.lower())

  # print (json.dumps(resultParsed[responseKeyword], indent=2, sort_keys=True))

  for obj in resultParsed[responseKeyword][keyword]:
    if (obj['level'] != 'INFO'):
      print(obj['created'])
      print("Domain: {} / Account: {}".format(obj['domain'], obj['account']))
      print(obj['description'])
      print('--------------------------------------')

  print("")


def getHealth_05 (keyword, command):
  print ('[{}]'.format(keyword))

  req = getReq (command)
  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  responseKeyword = "{}response".format(command.lower())

  # print (json.dumps(resultParsed[responseKeyword], indent=2, sort_keys=True))

  for num, obj in enumerate(resultParsed[responseKeyword][keyword], start=1):
    print(obj['sent'])
    print(obj['name'])
    print(obj['description'])
    print('--------------------------------------')

    if (num == int(sys.argv[1])):
      break

  print("")


def getHealth_06 (keyword, command):
  print ('[{}]'.format(keyword))

  req = getReq (command)
  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  responseKeyword = "{}response".format(command.lower())

  # print (json.dumps(resultParsed[responseKeyword], indent=2, sort_keys=True))

  for obj in resultParsed[responseKeyword][keyword]:
    if (obj['type'] == 'Routing'):

      cpuAvail = 100-float(obj['cpuused'][:-1])
      memTotalGB = float(obj['memorytotal']) / 1024 / 1024 / 1024
      memTotalTB = memTotalGB / 1024
      memAvailGB = (float(obj['memorytotal']) - float(obj['memoryused'])) / 1024 / 1024 / 1024
      memAvailTB = memAvailGB / 1024

      print("{}".format(obj['name']))
      print("CPU Avail: {:.2f} %".format(cpuAvail))
      print("Memory Total: {:.2f} GB / {:.2f} TB".format(memTotalGB, memTotalTB))
      print("Memory Avail: {:.2f} GB / {:.2f} TB".format(memAvailGB, memAvailTB))
      print('--------------------------------------')

  print("")


def getHealth_07 (keyword, command):
  print ('[{}]'.format(keyword))

  req = getReq (command)
  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  responseKeyword = "{}response".format(command.lower())

  # print (json.dumps(resultParsed[responseKeyword], indent=2, sort_keys=True))

  for obj in resultParsed[responseKeyword][keyword]:
    print("{}".format(obj['tags']))
    print("Disk Total: {:.2f} TB".format(float(obj['disksizetotal']) / 1024 / 1024 / 1024 / 1024))
    print("Disk Avail: {:.2f} TB".format((float(obj['disksizetotal']) - float(obj['disksizeallocated'])) / 1024 / 1024 / 1024 / 1024 ))
    print('--------------------------------------')


def getHealth_08 (command):
  print ('secondary')

  req = getReq (command)
  res = urllib2.urlopen(req)
  result = res.read()

  resultParsed = json.loads (result)

  # print (json.dumps(resultParsed, indent=2, sort_keys=True))

  for obj in resultParsed['listcapacityresponse']['capacity']:
    if (obj['type'] == 6):
      print ("Name: {}".format(obj['name']))
      print ("Zone Name: {}".format(obj['zonename']))
      print ("Disk Total: {:.2f} GB".format(float(obj['capacitytotal']) /1024 / 1024 / 1024))
      print ("Disk Avail: {:.2f} GB".format( (float(obj['capacitytotal'] - float(obj['capacityused']))) / 1024 / 1024 / 1024))
      print('--------------------------------------')

  print ("")

getHealth_01 ('zone', 'listZones')
getHealth_01 ('pod', 'listPods')
getHealth_01 ('cluster', 'listClusters')
getHealth_01 ('storagepool', 'listStoragePools')
getHealth_01 ('systemvm', 'listSystemVms')
#getHealth_01 ('router', 'listRouters')
#getHealth_01 ('network', 'listNetworks')

getHealth_02 ('host', 'listHosts')
#getHealth_03 ('virtualmachine', 'listVirtualMachines')
getHealth_04 ('event', 'listEvents')
getHealth_05 ('alert', 'listAlerts')
getHealth_06 ('host', 'listHosts')
getHealth_07 ('storagepool', 'listStoragePools')
getHealth_08 ('listCapacity')


print("[Info]")
print("cloudstack main dashboard / vm status / network / vpc / virtualrouter / check needed!")
