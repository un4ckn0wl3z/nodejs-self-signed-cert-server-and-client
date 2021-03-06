# Self Signed Certificates and Certifcate Chains

Enterprises might often deal with internal self signed
certificates. Also testing.

I've found it unnecessarily complicated to find the answers to how to
do this.

So here's a working demo.

Just:

```
npm install
npm test
```

and a demonstration will be performed.

For those curious, the steps should be, first:

```bash
echo 01 > cacert.srl
openssl genrsa -out ca.key 2048
# The domain here cannot be localhost
openssl req -x509 -new -nodes -key ca.key -days 365 -out cacert.pem -subj "/CN=example.com"
```

to generate the signed CA cert.

Then this bash:

```bash
openssl genrsa -out my.key 2048
openssl req -new -key my.key -out csr -subj "/CN=localhost"
openssl x509 -req -in csr -CA cacert.pem -CAkey ca.key -days 365 -out cert.pem
```

to generate a cert signed by that CA.

Then a server can have the cert and key:

```javascript
let opts = { 
  key: await fs.promises.readFile("my.key"),
  cert: await fs.promises.readFile("cert.pem")
};
https.createServer(opts, app).listen(2443);
```

and a request object can have the CA:

```javascript
 let opts = {
    method: 'GET',
    hostname: "localhost",
    port: listener.address().port,
    path: '/',
    ca: await fs.promises.readFile("cacert.pem")
  };

  https.request(opts, (response) => { }).end();
```

and that's it.

## Using fetch

Fetch under node is a bit different. With node-fetch, for example, you
need to use an Agent for fetch:

```javascript
let response = await fetch("https://localhost:2443", {
  agent: new https.Agent({ ca: ca })
});
let fetchHtml = await response.text();
```

```
https://stackoverflow.com/questions/45088006/nodejs-error-self-signed-certificate-in-certificate-chain/45088585
```
