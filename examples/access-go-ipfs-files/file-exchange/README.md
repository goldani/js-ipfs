# Accessing go-ipfs files from the browser

**WIP**

InterPlanetary File Exchange enables you to share files with others via IPFS. 

IPFE was built to demonstrate interoperability between the go-ipfs and js-ipfs (browser and Node.js) instances.

## TODO

- Need to merge https://github.com/ipfs/browserify-zlib-next/pull/23 and use the updated module. Otherwise the UI will error on zlib._transform.
- Make sure go-ipfs and the UI connect to each other (last time I checked they didn't)
- Use new signal server address everywhere (src/App.js, ipfe-daemon.js, ipfe-add.js)
- ipfe-add currently uses js-ipf(node.js) daemon. But in order to add files from go-ipfs daemon, it would need to use js-ipfd-ctl. A quick way would be to use ipfs-daemon (I would *highly* recommend to do so), otherwise need to add the init dance for js-ipfsd-ctl much like `src/ipfs.js`.

## Requirements

- Node.js >= v4.x
- Npm >= v3.x

## Step-by-step Instructions

### Start a go-ipfs daemon

1. Install go-ipfs from master (TODO: link). 

2. Run `ipfs init`

3. Edit your IPFS config file, located at `~/.ipfs/config`

4. Add a Websocket listener address to `Addresses.Swarm`. It should look like this after editing:
```
"Addresses": {
  "API": "/ip4/127.0.0.1/tcp/5001",
  "Gateway": "/ip4/0.0.0.0/tcp/8080",
  "Swarm": [
    "/ip4/0.0.0.0/tcp/4001",
    "/ip4/0.0.0.0/tcp/9999/ws"
  ]
},
```

5. Start the go-ipfs daemon with:
```
ipfs daemon
```

6. You should see the Websocket address in the output:
```
Initializing daemon...
Swarm listening on /ip4/127.0.0.1/tcp/4001
Swarm listening on /ip4/127.0.0.1/tcp/9999/ws
Swarm listening on /ip4/192.168.10.38/tcp/4001
Swarm listening on /ip4/192.168.10.38/tcp/9999/ws
API server listening on /ip4/127.0.0.1/tcp/5001
Gateway (readonly) server listening on /ip4/0.0.0.0/tcp/8080
Daemon is ready
```

If you see address like `Swarm listening on /ip4/127.0.0.1/tcp/9999/ws`, it means all good!

### Start an InterPlanetary File Exchange daemon

1. Install the project's dependencies:
```
npm install
```

2. Start the browser app with:
```
npm start
```

This will open the app in your browser at http://localhost:3000/.

3. In the browser app, open a file exchange feed url, eg. http://localhost:3000/hello-world.

4. Start the Node.js ipfe-daemon with:
```
node ipfe-daemon hello-world
```

The first argument after `ipfe-daemon` is the name of the file exchange feed.

5. In the browser app, open the Peers view by clicking on "Searching for peers..." or "1 peer". You will see which peers you're connected to.

6. Now go back to the terminal and find the Websocket multiaddress of your go-ipfs daemon by running:
```
ipfs id
```

This will output all the address your go-ipfs daemon is listening at. Find the one with `ws` in it, and the port you added to the go-ipfs configuration file. It looks something like this:
```
/ip4/127.0.0.1/tcp/9999/ws/ipfs/QmZGH8GeASSkSZoNLPEBu1MqtzLTERNUEwh9yTHLEF5kcW
```

7. Copy the address and paste it into the "Connect" input field in the browser app and press "Connect".

8. You should now see the two IPFS instances connected. You can verify it by observing the Peer view in the browser app and running the following command in the terminal:
```
ipfs swarm peers
```

### Add files to InterPlanetary File Exchange

1. Add a file by running:
```
node ipfe-add hello-world ipfe-add.js
```

You should see the following output:
```
Starting...
Swarm listening on /libp2p-webrtc-star/ip4/127.0.0.1/tcp/9090/ws/ipfs/QmYRSN9BdRU5oYM1ryAWF518ogv8SkwZyux4aEWpxoaYZA
IPFS node is ready
New peers for 'hello-world':
QmRNhXuS78LtUVLdJUJKmeLfE7K64CGBmwCiXY1sgJ6NaV
------------------------------
File | Hash | Size | Mime Type
------------------------------
ipfe-add.js | Qmdp8yhkyNGeqEYpkpqAm7duYqsEiJUi3KXfUFuUcyR2GR | 2588 | application/javascript
```

Note that the multihash at the end of the swarm address will be different.

6. Once you see the output above, it means the file was added successfully!

7. Now go back to the browser app and observe the file in the list of files.

8. Click the file to open it.

9. You have successfully added a file in go-ipfs and downloaded it to the browser.

You can also add files to the browser app by dragging and dropping them. Once you do so, you should see the updated file list in the terminal running `ipfe-daemon`.
