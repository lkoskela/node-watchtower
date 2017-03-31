# node-watchtower
A docker image updater inspired by [v2tec/watchtower](https://github.com/v2tec/watchtower), implemented by Node.js.

## Features
- Auto check if there is image update for each running container periodically.
- You decide whether to apply the update or not.
- Auto fall back to previous working version if the updated container failed to start.
- Load image from a gzipped tarball file or a Readable stream.
- Push local image to the private registry server.

## Usage
```
npm install node-watchtower
```

### Code
```js
import Watchtower from 'node-watchtower';

/* Create a watchtower instance */
const watchtower = new Watchtower({
  checkUpdateInterval: 180, /* 3 mins, set to 0 to disable update check */
  timeToWaitBeforeHealthyCheck: 10, /* 10 seconds to wait for the updated container to start */
});

/* Add your own private registry server */
watchtower.addRegistryAuth('your.own.server:5000', {
  username: 'csy',
  password: 'chardi',
  auth: '',
  email: '',
});

/* Listen to updateFound event */
watchtower.on('updateFound', (containerInfo) => {
  /* Apply update immediately */
  watchtower.applyUpdate(containerInfo).then((updatedContainerInfo) => {
    console.log(`Container ${updatedContainerInfo.Name} is updated`);
  }).catch((error) => {
    console.log(`Container ${containerInfo.Name} update failed.`, error.message);
  });
});

/* Activate the watchtower. This will start checking updates for every 3 mins */
watchtower.activate();

/* Load docker image from a file */
watchtower.load(`./test/images/alpine-3.5.tar.gz`).then((repoTags) => {
  console.log(`Image ${repoTags} loaded.`);
}).catch((error) => {
  console.log('Failed to load image.', error.message);
});

/* Push a image with repo:tag to the registry server */
watchtower.push('your.own.server:5000/alpine:3.5').then(() => {
  console.log('Image pushed');
}).catch((error) => {
  console.log('Image push failed.', error.message);
});

/* Inactivate the watchtower */
watchtower.inactivate().then(() => {
  console.log('Watchtower inactivated');
});

```

### Run as web service
Refer to [express-watchtower](https://github.com/csy1983/express-watchtower).

## TBD
- Push image to docker hub.
