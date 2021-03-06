# predix-fast-token
Node module to verify UAA tokens used when protecting REST endpoints

## Usage
Install via npm

```
npm install --save predix-fast-token
```

Basic usage with a JWT token and list of trusted issuers

```javascript
const pft = require('predix-fast-token');
pft.verify(token, trusted_issuers).then((decoded) => {
     // The token is valid, not expired and from a trusted issuer
     // Use the value of the decoded token as you wish.
     console.log('Good token for', decoded.user_name);
}).catch((err) => {
    // Token is not valid, or expired, or from an untrusted issuer.
    console.log('No access for you', err);
});
```

As an expressjs middleware

```javascript
'use strict';
const express = require('express');
const bearerToken = require('express-bearer-token');
const predixFastToken = require('predix-fast-token');
const app = express();

const trusted_issuers = ['https://example.uaa.predix.io/oauth/token', 'https://another.uaa.predix.io/oauth/token'];

app.get('/hello', (req, res, next) => {
    res.send('Howdy my unsecured friend!');
});

// Ensure Authorization header has a bearer token
app.all('*', bearerToken(), function(req, res, next) {
    console.log('Req Headers', req.headers);
    if(req.token) {
        predixFastToken.verify(req.token, trusted_issuers).then((decoded) => {
            req.decoded = decoded;
            console.log('Looks good');
            next();
        }).catch((err) => {
            console.log('Nope', err);
            res.status(403).send('Unauthorized');
        });
    } else {
		console.log('Nope, no token');
        res.status(401).send('Authentication Required');
    }
});

app.get('/secure', (req, res, next) => {
    res.send('Hello ' + req.decoded.user_name + ', my authenticated chum!');
});

// Need to let CF set the port if we're deploying there.
const port = process.env.PORT || 9001;
app.listen(port);
console.log('Started on port ' + port);

```
