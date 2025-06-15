# github webhooks

go to `https://github.com/{USER}/{REPO}/settings/hooks` and enable webhook

then host a server listening for push events, eg node:

```javascript
import { createServer } from 'http';
import { exec } from 'child_process';

const server = createServer((req, res) => {
  if (req.headers['x-hub-signature-256'] === undefined) {
    res.writeHead(405, { 'Content-Type': 'text/plain' });
    res.end('method not allowed');
  }
  if (req.method === 'POST') {
    req.on('data', _ => { });
    req.on('end', () => {
      console.log('received push');
      exec('git pull', (error) => {
        if (error) {
          console.error('error: ', error);
        } else {
          console.log('pulled');
        }
      });
      res.writeHead(200);
      res.end('ok');
    });
  } else {
    res.writeHead(405, { 'Content-Type': 'text/plain' });
    res.end('method not allowed');
  }
});

server.listen(4567, () => {
  console.log('listening on 4567 for push events');
});
```
