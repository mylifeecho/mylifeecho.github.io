---
layout: post
title: "PeerJS + ASP.NET"
description: ""
category: dev
tags: [ PeerJS, Node.js, iisnode, ASP.NET, peer-2-peer ]
---
{% include JB/setup %}

Progress in Web development astonishes. We spend less time using desktop applications and more time using only browser. There are powerful web tools like Google Drive and Office Live, online dictionaries and translators, messengers, graphics editors, 3D games and even IDEs. You need just have the Internet and modern browser. More and more features constantly appear. And one of them is peer-to-peer networking between browsers. Now I’m going to show how you can add peer-2-peer interaction between clients into your ASP.NET application with using PeerJS step by step.

PeerJS is amazing javascript library contains two parts: client-side and server-side written for Node.js. It wraps WebRTC implementation API and helps to easily establish p2p communication between browsers. I recommend you to play with it on [PeerJS site][peerjs], read good [Get Started][peerjs-gs] and [API Documantation][peerjs-api] pages and [try few demos][peerjs-demo]. There is a cloud server for establishing peer-to-peer communication which you can use for free for up to 50 concurrent clients but if you think that you need your own server in context of ASP.NET application I hope this post would be interest for you.

I will use example of [advanced p2p][peerjs-chat] chat from demo page of PeerJS because client side is one and the same for ASP.NET, Ruby on Rails or PHP, but it will be changed a bit to show situation when we already have ASP.NET Web application and you need to add and use own PeerJS server hosted on IIS.

## Install PeerJS Server

PeerJS server is opensource and written for Node.js. IIS does not support Node.js out from the box, but there is Node.js handler for IIS named **iisnode** so you have to install it. The most easier way to do it with help of Web Platform Installer, but of course you can download and install it manually from here. More information about iisnode and detailed explanation why you may want to use Node.js on IIS you can find in post written by Scott Hanselman

![Install IISNode][iisnode-inst]

After installation follow to the PeerJS server page on GitHub and download files from lib directory to for example Node folder. This files contain PeerServer module which we will use in our web application. Let’s create app.js and put a following code there:

```js
var fs = require('fs');
var PeerServer = require('./server').PeerServer;

var server = new PeerServer({ port: 9000 });
```

There are two differences from example in PeerJS readme: we have changed **./server** argument to load local version of module (not installed with npm) and skip passing SSL certificate for our test project. fs module could be used here to set SSL certificate or some configuration from file.

![App in Visual Studio][server-app]

PeerJS server has dependencies from ws (work with WebSockets) and restify (REST api) packages which might need to install using npm package manager. I did it manually before start the server and got errors during installation of restify package, solved problems one by one, but when npm asked me to install .NET 3.5, I stopped it and was surprised that server works despite the problem with installation one of its dependency.

The last thing we have to do to start server is add iisnode handler configuration to our Web.config and specify path to the PeerJS application.

```xml
<system.webserver>
  <handlers>
    <add name="iisnode" path="Node/app.js" verb="*" modules="iisnode" />
  </handlers>
</system.webserver>
```

Open your IIS Manager and follow to the the Handlers mappings section. You should see something like this. 

![NodeJS handler in IIS][iisnode]

After it if 9000 port is free, server starts and we can try to follow the link `http://localhost:9000/someid/id`. If everything is ok we will receive id from our server. 

## Client Side

Now we need to do small changes in code of our chat application. To play with server you can publish your web application to [AppHarbor][ah]. They [announced][ah-ws] support of node.js and web sockets which are required for p2p interaction.

So I am going to launch ASP.NET application with PeerJS server in my Virtual Environment with Windows 8 (as I know there is an issue with web sockets support on Windows 7) and the first browser will be on virtual machine, but the second browser will be on host machine. So I set IP of my virtual machine here:

```js
var peer = new Peer({ host: '192.168.137.130', port: 9000 });
```

This is one difference from original chat example on PeerJS website. 

Let’s play with it.

Browsers connect to server, get unique Ids and one user can connect to another by this id. After it they can send messages and files to each other. Below you can see browser on host machine on the left side and browser and IIS Manager in virtual machine on the right side.

![Chat online][test-online]

Clients are connected and can interact, but what is happens if we shutdown the server?

![Chat offline][test-offline]

Yes, it’s true p2p. Server is offline but clients are able to interact to each other 

## Summary

WebRTC is being implemented as well as work on PeerJS is in progress. It is pretty raw but I believe it will be ready soon. Once WebRTC standard is implemented it will be really amazing. There are a few known issues and restrictions with PeerJS at the moment of writing this post.

DataChannel which uses for peer-2-peer communication is still implementing and available in stable version of Google Chrome 26+ and Nightly build of Firefox 22+
Firefox and Chrome are not compatible yet, so you cannot establish p2p connection between them.
You have to provide your own TURN server for compatibility with symmetric NATs. By default the STUN server provided by Google is used.
About detailed and actual status of relations between WebRTC and PeerJS you can find here.

## References

* [https://github.com/mylifeecho/peerjsapp](https://github.com/mylifeecho/peerjsapp) - source code from this post
* [http://peerjs.com][peerjs] - PeerJS site
* [http://peerjs.com/peerserver](http://peerjs.com/peerserver) - cloud PeerJS server
* [https://github.com/peers/peerjs-server](https://github.com/peers/peerjs-server) - source code of PeerJS Server

[peerjs]: http://peerjs.com
[peerjs-gs]: http://peerjs.com/start
[peerjs-api]: https://github.com/peers/peerjs/blob/master/docs/api.md
[peerjs-demo]: https://github.com/peers/peerjs/tree/master/examples
[peerjs-chat]: http://cdn.peerjs.com/demo/chat.html
[ah]: http://appharbor.com
[ah-ws]: http://blog.appharbor.com/2013/02/19/websocket-support-for-net-and-node-js-apps

[iisnode-inst]: {{ BASE_PATH }}/assets/posts/2013-06-23/iisnode_installation.png
[server-app]: {{ BASE_PATH }}/assets/posts/2013-06-23/server_app.png
[iisnode]: {{ BASE_PATH }}/assets/posts/2013-06-23/iisnode.png
[test-online]: {{ BASE_PATH }}/assets/posts/2013-06-23/test_online.png
[test-offline]: {{ BASE_PATH }}/assets/posts/2013-06-23/test_offline.png