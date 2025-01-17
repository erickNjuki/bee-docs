---
title: Upload and Download Files
id: upload-and-download
---


import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

When you upload your files to the swarm, they are split into 4kb
_chunks_ and then distributed to nodes in the network that are
responsible for storing and serving these parts of your content. Each
chunk has a _postage stamp_ stuck to it which attaches a value in xBZZ
to that chunk which you agree to _burn_ when buying the batch of stamps. This
signifies to storage nodes that this data is important, and supposed
to be retained in the Distributable Immutable Store of Chunks
(_DISC_).

## Overview

To upload data to the swarm, you must perform the following steps:

1. Fund your node's wallet with xBZZ.
2. Purchase a _batch_ of stamps with your xBZZ.
3. Wait for the batch to propogate into the network.
4. Upload your content, specifying the _batch id_ so that Bee can attach stamps to your chunks.
5. Download your content using your content's hash.

## Purchasing Your Batch of Stamps

In order to upload your data to swarm, you must agree to burn (spend)
some of your xBZZ to signify to storer and fowarder nodes that this
content is valued. Before you proceed to the next step, you must buy
stamps! See this guide on how to [purchase an appropriate batch of
stamps](/docs/develop/access-the-swarm/buy-a-stamp-batch).

### Upload

Once your Bee node is running, a HTTP API is enabled for you to interact with. The command line utility [curl](https://ec.haxx.se/http/http-multipart) is a great way to interact with a Bee node's API. Swarm CLI alternative commands are also included as a more user-friendly way of interacting with your Bee node's API.




<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Swarm CLI', value: 'swarm-cli'},
]}>
<TabItem value="api">

#### API

First, let's check to see if the API is running as expected...

```bash
curl http://localhost:1633
```

```
Ethereum Swarm Bee
```

Once running, a file can be uploaded by making an HTTP POST request to the `files` endpoint of the Bee API.

Here, you must specify your _Batch ID_ in the `Swarm-Postage-Batch-Id` header as follows.

```bash
curl --data-binary "@bee.jpg" -H "Swarm-Postage-Batch-Id: 78a26be9b42317fe6f0cbea3e47cbd0cf34f533db4e9c91cf92be40eb2968264"   http://localhost:1633/bzz
```

We may also pass the appropriate mime type in the `Content-Type` header, and a file name to the `name` query parameter so that the file will be correctly handled by web browsers and other applications.

```bash
curl --data-binary "@bee.jpg" -H "Swarm-Postage-Batch-Id: 78a26be9b42317fe6f0cbea3e47cbd0cf34f533db4e9c91cf92be40eb2968264" -H "Content-Type: image/jpg" http://localhost:1633/bzz
```

:::danger
Data uploaded to the swarm is always public. In Swarm, sensitive files
must be [encrypted](/docs/develop/access-the-swarm/store-with-encryption)
before uploading to ensure their contents always remains private.
:::

When succesful, a JSON formatted response will be returned, containing
a **swarm reference** or **hash** which is the _address_ of the
uploaded file, for example:

```json
{
  "reference": "22cbb9cedca08ca8d50b0319a32016174ceb8fbaa452ca5f0a77b804109baa00"
}
```

Keep this _address_ safe, as we'll use it to retrieve our content later on.

</TabItem>

<TabItem value="swarm-cli">

#### Swarm CLI
We have a `test.txt` file we wish to upload, let's check its contents.

```bash
cat test.txt
This is a test file
It will be used to test uploading and downloading from Swarm
```

Check that our node is operating normally.  

```bash
swarm-cli status
```

```bash
Bee
API: http://localhost:1633 [OK]
Debug API: http://localhost:1635 [OK]
Version: 1.17.5-50fcec7b
Mode: full

Topology
Connected Peers: 175
Population: 13614
Depth: 9

Wallet
xBZZ: 85.5638752768932272
xDAI: 4.753393401487287091

Chequebook
Available xBZZ: 0.0000000000018
Total xBZZ: 0.0000000000018

Staking
Staked BZZ: 10

Redistribution
Reward: 831386836533248000
Has sufficient funds: true
Fully synced: true
Frozen: false
Last selected round: 202266
Last played round: 202266
Last won round: 186776
Minimum gas funds: 101250000000000000
```

List our stamps.

```bash
swarm-cli stamp list
```

Copy the ID of the stamp we wish to use.

```bash
Stamp ID: daa8c5b36e1cf481b10118a8b02430a6f22618deaa6ba5aa4ea660de66aa62db
Usage: 7%
Remaining Capacity: 7.50 GB
TTL: 91 days 1 hour 45 minutes 28 seconds
Expires: 2024-02-01
```
Use the stamp ID to upload our file.

```bash
swarm-cli upload test.txt --stamp daa8c5b36e1cf481b10118a8b02430a6f22618deaa6ba5aa4ea660de66aa62db
```

If successful, we will receive the hash of the uploaded file and the URL where it is reachable.

```bash
Swarm hash: 1ffd2b67c8f34596a0b8375be29423c2d47e7995fcac8dd83fbd34e3d839b5a2
URL: http://localhost:1633/bzz/1ffd2b67c8f34596a0b8375be29423c2d47e7995fcac8dd83fbd34e3d839b5a2/
Stamp ID: daa8c5b3
Usage: 7%
Remaining Capacity: 7.50 GB 
```

Let's check that the file is downloadable.

```bash
swarm-cli download 1ffd2b67c8f34596a0b8375be29423c2d47e7995fcac8dd83fbd34e3d839b5a2
test.txt OK
```

And let's confirm that the contents of the file are correct.

```bash
cat test.txt
This is a test file
It will be used to test uploading and downloading from Swarm
```
</TabItem>
</Tabs>



In Swarm, every piece of data has a unique _address_ which is a unique and reproducible cryptographic hash digest. If you upload the same file twice, you will always receive the same hash. This makes working with data in Swarm super secure!

:::info
If you are uploading a large file it is useful to track the status of your upload as it is processed into the network. To improve the user experience, learn how to [follow the status of your upload](/docs/develop/access-the-swarm/syncing).

Once your file has been **completely synced with the network**, you will be able to turn off your computer and other nodes will take over to serve the data for you!
:::

## Download

Once your file is uploaded to Swarm it can be easily downloaded. 


<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Swarm CLI', value: 'swarm-cli'},
]}>
<TabItem value="api">

#### API

Uploaded files can be retrieved with a simple HTTP GET request.

Substitute the _hash_ in the last part of the URL with the reference
to your own data.

```bash
curl -OJL http://localhost:1633/bzz/042d4fe94b946e2cb51196a8c136b8cc335156525bf1ad7e86356c2402291dd4/
```

You may even simply navigate to the URL in your browser:

[http://localhost:1633/bzz/22cb...aa00](http://localhost:1633/bzz/22cbb9cedca08ca8d50b0319a32016174ceb8fbaa452ca5f0a77b804109baa00)


</TabItem>

<TabItem value="swarm-cli">

#### Swarm CLI
Simply use the `swarm-cli download` command followed by the hash of the file you wish to download. 

```bash
swarm-cli download 1ffd2b67c8f34596a0b8375be29423c2d47e7995fcac8dd83fbd34e3d839b5a2
test.txt OK
```
And let's print out the file contents to confirm it was downloaded properly.

```bash
cat test.txt
This is a test file
It will be used to test uploading and downloading from Swarm
```
</TabItem>
</Tabs>



## Public Gateways

To share files with someone who isn't running a Bee node yet, simply change the host in the link to be one of our public gateways. Send the link to your friends, and they will be able to download the file too!

[https://download.gateway.ethswarm.org/bzz/22cb...aa00/](https://download.gateway.ethswarm.org/bzz/22cbb9cedca08ca8d50b0319a32016174ceb8fbaa452ca5f0a77b804109baa00/)

<!-- If you are unable to download your file from a different Bee node, you may be experiencing connection issues, see [troubleshooting connectivity](/docs/troubleshooting/connectivitiy) for assistance. -->
