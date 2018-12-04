# node-red-contrib-udpdnserver

This is a fully formed UDP Node-Red node that provides a caching DNS Server.

The Node uses an alaSQL in memory database that can store and then cache requests to significantly speed up local server requests, for example requests can go from 350ms to 8ms when cached.

Cache values (on a per domain basis) for time to live (TTL) are in seconds, after that it deletes the entry(s) and starts learning again.

The Cache learning algorythm (LTTL) is in seconds, it allows (on a per domain basis) the ability to learn about multiple addresses or requests, for example, if the LTTL is set to 60, and multiple command line "nslookup www.google.com" requests are performed by the user, then all of the addresses received by the cache are randomly sent back to the user on susequent cached requests, this is done until the domain TTL has expired and it then flushes the RR (Resource Request) cache and the learning is started again.

Note all RR types are intercepted and then made available to the cache,user's can decide to further process them in their own Node-Red processes and/or cache.

One designed capabilty is that the Node is programmable in the Node-Red builder, it allows users to add alaSQL SQL/Javascript functions and tables - functionality to extend the Node.

The first output is from the decision engine eg: if the result is from the cache or from the upstream DNS resolver, you can add as many upstream resolvers as you like, a random selection is applied select an upstream DNS resolver.
Note that upstream resolvers can have a different port number and this can be set by way of addrress and port for example

msg1.sql="select uds_insertUpstreamAddress('8.8.4.4',5353) rr";

msg1.sql="select uds_insertUpstreamAddress('8.8.4.4',53) rr";

msg1.sql="select uds_insertUpstreamAddress('8.8.4.4') rr";    // use default 53

This allows you to cascade node-red-contrib-updnserver nodes (only available using multiple upstream node-red or bind instances)

#Zones
```
Zones are now provided by a new zones table and subnets are loaded into it eg: local,network,dmz... an unlimitied number can be created, including an unlimited of subnets can be defined.
Subnets are resolved to zoneNames during the resolving process meaning, the incoming IP calling address can now have its own address cached and a lookup on a per zone is now performed.
```

Another capability that it has, is it allows tracking domains that requests have gone to, also, it tracks the calling address and ports the requests originated from, this is provided by the second output.

Note that the input and third output are for requests and responces to alaSQL and allow you to easily extend the Node.

Currently ONLY IPV4 requests are resolved, although the node itself actually stores "Type 28" request for IPV6, I just havent written the functions yet.

Refer to this wiki for the RR types. https://en.wikipedia.org/wiki/List_of_DNS_record_types

#To Install
```
npm install node-red-contrib-udpdnserver
Import the flow below into Node-Red
set the ip address of the listener in the node eg 127.0.0.1 
set the port if not on the default port (53)
point resolv.conf to this address
Try it out
```
Use this flow to get you started, note that on the info tab, the basic node standard functions are and will be maintained.
--------------------------------------------------------------------------------------------------------------------------
```
[{"id":"b4ccc63d.b18158","type":"tab","label":"DNS Server Test","disabled":false,"info":""},{"id":"580cabae.190944","type":"switch","z":"b4ccc63d.b18158","name":"","property":"payload.result","propertyType":"msg","rules":[{"t":"eq","v":"resolved","vt":"str"},{"t":"eq","v":"cached","vt":"str"},{"t":"else"}],"checkall":"false","repair":false,"outputs":3,"x":790,"y":160,"wires":[["e3c3cfb8.76e208"],["35ca5018.f41438"],[]]},{"id":"b6d13884.c9c088","type":"debug","z":"b4ccc63d.b18158","name":"RAW Answer","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","x":810,"y":220,"wires":[]},{"id":"35ca5018.f41438","type":"debug","z":"b4ccc63d.b18158","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","x":930,"y":160,"wires":[]},{"id":"c54aa834.8bd038","type":"switch","z":"b4ccc63d.b18158","name":"","property":"payload.sql","propertyType":"msg","rules":[{"t":"cont","v":"CREATE DATABASE","vt":"str"},{"t":"cont","v":"getIPV4Address","vt":"str"},{"t":"cont","v":"getIPV4cName","vt":"str"},{"t":"cont","v":"delete from","vt":"str"},{"t":"cont","v":"CACHED RESULT","vt":"str"},{"t":"cont","v":"select * from learnrr","vt":"str"},{"t":"cont","v":"rrParse","vt":"str"},{"t":"cont","v":"rrInsert","vt":"str"},{"t":"cont","v":"checkLearn","vt":"str"},{"t":"cont","v":"checkCache","vt":"str"},{"t":"cont","v":"insertUpstreamAddress","vt":"str"},{"t":"cont","v":"select * from rr","vt":"str"},{"t":"else"}],"checkall":"false","repair":false,"outputs":13,"x":570,"y":400,"wires":[["f4066ed9.43246"],["83aa2f3f.ffb2"],["3a76bd31.f7d3a2"],["9eeb4658.9d1de"],["9eeb4658.9d1de"],["7c76cda5.a8ea44"],["1c829cc4.292e43"],[],[],["3cf0ef91.02bd6"],[],["8cdd37b3.46a07"],["8e29ef52.b05748"]]},{"id":"7c76cda5.a8ea44","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload.sqlResult","x":840,"y":420,"wires":[]},{"id":"9eeb4658.9d1de","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","x":790,"y":380,"wires":[]},{"id":"cdb0e58e.0baf78","type":"function","z":"b4ccc63d.b18158","name":"Who's learning","func":"var msg1 = {};\nmsg1.sql=\"select * from learnrr rr\";\n//msg1.sql=\"SELECT * from rr \";\nreturn msg1;","outputs":1,"noerr":0,"x":320,"y":380,"wires":[["5673e96a.fff1f8"]]},{"id":"c3548720.e013c","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":120,"y":380,"wires":[["cdb0e58e.0baf78"]]},{"id":"3a76bd31.f7d3a2","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload.sqlResult[0].rr","x":860,"y":340,"wires":[]},{"id":"e40ac9c1.cdd6a","type":"function","z":"b4ccc63d.b18158","name":"flush rr learn","func":"var msg1 = {};\nmsg1.sql=\"delete from learnrr\";\n//msg1.sql=\"SELECT * from rr \";\nreturn msg1;","outputs":1,"noerr":0,"x":310,"y":420,"wires":[["5673e96a.fff1f8"]]},{"id":"10beb718.5c68b1","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":120,"y":420,"wires":[["e40ac9c1.cdd6a"]]},{"id":"8e29ef52.b05748","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","x":790,"y":580,"wires":[]},{"id":"eba9c68.48e3bb8","type":"function","z":"b4ccc63d.b18158","name":"show the rr list","func":"var msg1 = {};\nmsg1.sql=\"select * from rr\";\n//msg1.sql=\"SELECT * from rr \";\nreturn msg1;","outputs":1,"noerr":0,"x":320,"y":340,"wires":[["5673e96a.fff1f8"]]},{"id":"1669dde0.d9927a","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":120,"y":340,"wires":[["eba9c68.48e3bb8"]]},{"id":"1388daa2.87e14d","type":"function","z":"b4ccc63d.b18158","name":"checkLearnRrCache","func":"var msg1 = {};\nmsg1.sql=\"select uds_checkLearn() rr\";\nreturn msg1;","outputs":1,"noerr":0,"x":340,"y":180,"wires":[["5673e96a.fff1f8"]]},{"id":"8672a975.df10a8","type":"function","z":"b4ccc63d.b18158","name":"get upstream Addr","func":"var msg1 = {};\nmsg1.sql=\"select uds_getUpstreamAddress() rr\";\n//msg1.sql=\"SELECT * from rr \";\nreturn msg1;","outputs":1,"noerr":0,"x":330,"y":300,"wires":[["5673e96a.fff1f8"]]},{"id":"4e213b48.58cac4","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":120,"y":300,"wires":[["8672a975.df10a8"]]},{"id":"80764d38.1bcd6","type":"function","z":"b4ccc63d.b18158","name":"insertUpstreamAddress","func":"var msg1 = {};\nmsg1.sql=\"select uds_insertUpstreamAddress('8.8.8.8') rr\";\n//msg1.sql=\"SELECT * from rr \";\nreturn msg1;","outputs":1,"noerr":0,"x":340,"y":220,"wires":[["5673e96a.fff1f8"]]},{"id":"90902ba1.6b7b1","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":true,"onceDelay":0.1,"x":130,"y":220,"wires":[["80764d38.1bcd6"]]},{"id":"757074d5.ec36ec","type":"function","z":"b4ccc63d.b18158","name":"checkRrCache","func":"var msg1 = {};\nmsg1.sql=\"select uds_checkCache() rr\";\nreturn msg1;","outputs":1,"noerr":0,"x":320,"y":140,"wires":[["5673e96a.fff1f8"]]},{"id":"b458a17a.e4bb","type":"inject","z":"b4ccc63d.b18158","name":"Scan Cache's","topic":"","payload":"","payloadType":"date","repeat":"5","crontab":"","once":false,"onceDelay":0.1,"x":140,"y":140,"wires":[["757074d5.ec36ec"]]},{"id":"3cf0ef91.02bd6","type":"debug","z":"b4ccc63d.b18158","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload.sqlResult[0].rr[1]","x":860,"y":500,"wires":[]},{"id":"c8c7c9b2.ef22e","type":"function","z":"b4ccc63d.b18158","name":"insertUpstreamAddress","func":"var msg1 = {};\nmsg1.sql=\"select uds_insertUpstreamAddress('9.9.9.9') rr\";\n//msg1.sql=\"SELECT * from rr \";\nreturn msg1;","outputs":1,"noerr":0,"x":340,"y":260,"wires":[["5673e96a.fff1f8"]]},{"id":"5611b091.8e3ef","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":true,"onceDelay":0.1,"x":130,"y":260,"wires":[["c8c7c9b2.ef22e"]]},{"id":"e3c3cfb8.76e208","type":"function","z":"b4ccc63d.b18158","name":"Create rr Record","func":"var msg1 = {};\nmsg.payload.rr.forEach(function(element) {\ndelete element[\"ttl\"];\nmsg1.sql=\"SELECT uds_rrInsert('\"+\n         element.name+\n         \"','\"+\n         JSON.stringify(element)+\n         \"','\"+ '600'+\n         \"','\"+ '60' +\n         \"');\";\nnode.send([{sql:msg1.sql}]); \n});","outputs":1,"noerr":0,"x":600,"y":140,"wires":[["5673e96a.fff1f8"]]},{"id":"5673e96a.fff1f8","type":"udpdnserver","z":"b4ccc63d.b18158","address":"127.0.0.1","port":53,"name":"UDPdnServer","functions":"\nCREATE DATABASE UDPdnServer;\nUSE DATABASE UDPdnServer;\nCREATE TABLE rr(domainName, rr,ttl);\nCREATE TABLE learnrr(domainName,lttl);\nCREATE TABLE upStreams;\n\nCREATE FUNCTION uds_insertUpstreamAddress AS ``function(address) {\n  let count = alasql('USE DATABASE UDPdnServer;SELECT address FROM upStreams WHERE address = ?',[address]);\n  let recFound = count[1].length;\n  if (recFound == 0) {\n    alasql('USE DATABASE UDPdnServer;INSERT INTO upStreams (address) VALUES (?);',[address]);\n  } \n  return count;\n}``;\n\nCREATE FUNCTION uds_getUpstreamAddress AS ``function() {\n  let data = alasql('USE DATABASE UDPdnServer;SELECT address from upStreams rec;');\n  let addresses = data[1];\n  let count = addresses.length;\n  if (count > 0) {\n      var min=0;\n      var max=count-1;\n      var index=Math.floor(Math.random() * (max - min + 1)) + min;\n      return addresses[index];  \n   }\nelse\n  return '1.1.1.1';\n}``;\n\nCREATE FUNCTION uds_checkCache AS ``function() {\n  var currentTs = Math.round(new Date().getTime()/1000);  \n  let count = alasql('USE DATABASE UDPdnServer;SELECT distinct domainName FROM rr where ttl < ?;',[currentTs] );\n  let recFound = count[1].length;\n  if (recFound) {\n  for (var i = 0; i < recFound; i++) { \n     var domainName = count[1][i].domainName;\n     alasql('USE DATABASE UDPdnServer;delete from rr where domainName = ?;',[domainName]);\n    }      \n  };\n  return count;\n}``;\n\nCREATE FUNCTION uds_checkLearn AS ``function() {\n  var currentTs = Math.round(new Date().getTime()/1000);  \n  let count = alasql('USE DATABASE UDPdnServer;SELECT * FROM learnrr;');\n  let recFound = count[1].length;\n  if (recFound) {\n  for (var i = 0; i < recFound; i++) { \n    if (currentTs > count[1][i].lttl) {\n       var domainName = count[1][i].domainName;\n       alasql('USE DATABASE UDPdnServer;delete from learnrr where domainName = ?;',[domainName]);\n        }\n    }      \n  };\n  return count;\n}``;\n\nCREATE FUNCTION uds_rrLearn AS ``function(domainName,lttl) {\n  let count = alasql('USE DATABASE UDPdnServer;SELECT domainName FROM learnrr WHERE domainName = ?;',[domainName]);\n  let recFound = count[1].length;\n  if (recFound == 0) {\n    alasql('USE DATABASE UDPdnServer;INSERT INTO learnrr (domainName,lttl) VALUES (?,?);',[domainName,lttl]);\n  } \n  return count;\n}``;\n\nCREATE FUNCTION uds_rrInsert AS ``function(domainName,rr,ttl,lttl) {\n  let count = alasql('USE DATABASE UDPdnServer;SELECT * FROM rr WHERE domainName = ? and rr = ?;',[domainName,rr]);\n  let recFound = count[1].length;\n  if (recFound == 0) {\n     var currentT = Math.round(new Date().getTime() / 1000);\n     var dttl = currentT+parseInt(ttl);\n     var dlttl= currentT+parseInt(lttl);\n     count= alasql('USE DATABASE UDPdnServer;INSERT INTO rr (domainName,rr,ttl) VALUES (?,?,?);',[domainName,rr,dttl]);\n     // Start the domain learning\n     let learnrr = alasql('SELECT uds_rrLearn(?,?) learnrec;',[domainName,dlttl]);\n  } \n  return count;\n}``;\n\nCREATE FUNCTION uds_rrParse AS ``function(domainName,typeIn,dataIn,checkLearnrr) {\n  var rrIn; \n  if (domainName === '') { rrIn= alasql('USE DATABASE UDPdnServer;SELECT rr FROM rr' );}\nelse \n  if (checkLearnrr) { rrIn= alasql('USE DATABASE UDPdnServer; SELECT rr FROM rr where domainName = ? and not exists (select domainName from learnrr where domainName = ?);',[domainName,domainName]);}\nelse\n   { rrIn= alasql('USE DATABASE UDPdnServer; SELECT rr FROM rr where domainName = \"'+ domainName+'\"');}\n  var outRR = [];\n  for (var i = 0; i < rrIn[1].length; i++) { \n    if (dataIn === '')  {outRR.push(JSON.parse(rrIn[1][i].rr));}\n  else    \n    if (JSON.parse(rrIn[1][i].rr)[typeIn] == dataIn) {outRR.push(JSON.parse(rrIn[1][i].rr));}\n  }\n  return {\"domainName\":domainName,\"rrcount\":i-1,\"rr\":outRR}; \n}``;\n\n\nCREATE FUNCTION uds_getIPV4Address AS ``function(domainName) {\n  let data = alasql('SELECT uds_rrParse(?,?,?,?) rec;',[domainName,'type','1',true]);\n  let addresses = data[0].rec.rr;\n  let count = addresses.length;\n  if (count > 0) {\n      var min=0;\n      var max=count-1;\n      var index=Math.floor(Math.random() * (max - min + 1)) + min;\n      return {\"domainName\":domainName,\"type\":\"1\",\"address\":addresses[index].address};  \n   }\nelse\n  return {\"domainName\":domainName,\"type\":\"1\",\"address\":\"none\"};\n}``;\n\nCREATE FUNCTION uds_getIPV4cName AS ``function(domainName) {\n  let savedName = domainName;\n  let data1 = '';\n  let notdone = true;\n  do {\n    data1 = alasql('SELECT uds_rrParse(?,?,?,?) rec;',[savedName,'type','5',true]);\n    if (data1[0].rec.rr[0] != null) {\n       savedName = data1[0].rec.rr[0].data;\n    }\n    else { notdone=false; }\n    }\n  while (notdone);\nreturn {\"domainName\":domainName,\"type\":\"5\",\"data\":savedName};\n}``;\n\nCREATE FUNCTION uds_resolveIPV4Address AS ``function(domainName) {\n  let data1 = alasql('SELECT uds_getIPV4cName(?) rec;',[domainName]);\n  domainName = data1[0].rec.data;\n  let data2 = alasql('SELECT uds_getIPV4Address(?) rec;',[domainName]);\n  return data2[0].rec;\n}``;\n ","x":600,"y":260,"wires":[["580cabae.190944"],["b6d13884.c9c088"],["c54aa834.8bd038"]]},{"id":"1c829cc4.292e43","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload.sqlResult[0].rr","x":860,"y":460,"wires":[]},{"id":"83aa2f3f.ffb2","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload.sqlResult[0].rr","x":860,"y":300,"wires":[]},{"id":"f4066ed9.43246","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","x":810,"y":260,"wires":[]},{"id":"b5b897a.dd21f68","type":"inject","z":"b4ccc63d.b18158","name":"Scan Cache's","topic":"","payload":"","payloadType":"date","repeat":"5","crontab":"","once":false,"onceDelay":0.1,"x":140,"y":180,"wires":[["1388daa2.87e14d"]]},{"id":"8cdd37b3.46a07","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload.sqlResult","x":840,"y":540,"wires":[]},{"id":"3e3c45e1.6b94f2","type":"dns","z":"b4ccc63d.b18158","name":"","method":"resolve","x":330,"y":560,"wires":[["e3e75733.48c7d8"]]},{"id":"d297b1ae.324c58","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"www.google.com.au","payloadType":"str","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":150,"y":540,"wires":[["3e3c45e1.6b94f2"]]},{"id":"e3e75733.48c7d8","type":"debug","z":"b4ccc63d.b18158","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","x":510,"y":560,"wires":[]},{"id":"94ec8e0b.ca56c8","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"www.ebay.com.au","payloadType":"str","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":140,"y":580,"wires":[["3e3c45e1.6b94f2"]]},{"id":"dfe1239d.666008","type":"function","z":"b4ccc63d.b18158","name":"flush rr cache","func":"var msg1 = {};\nmsg1.sql=\"delete from rr\";\n//msg1.sql=\"SELECT * from rr \";\nreturn msg1;","outputs":1,"noerr":0,"x":310,"y":460,"wires":[["5673e96a.fff1f8"]]},{"id":"99a612bf.6ba59","type":"inject","z":"b4ccc63d.b18158","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":120,"y":460,"wires":[["dfe1239d.666008"]]}]
```
