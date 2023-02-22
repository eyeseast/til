# Blocking (or allowing) IP addresses in Node.js

I have a large Express application where I needed to limit access to a set of internal IP ranges. This being Node.js, there are a lot of third-party libraries that will do this, of varying age and quality. There are bespoke IP address parsers and different flavors of middleware. As ever with Node, it's hard to know what's reliable.

Thankfully, it turns out Node can handle this without leaving the standard library, using the [`net.Blocklist`](https://nodejs.org/api/net.html#class-netblocklist) class. It builds a list of individual addresses, ranges and subnets, and then matches addresses against that list. It works with both IPv4 and IPv6.

Here's an example from the docs:

```js
const blockList = new net.BlockList();
blockList.addAddress("123.123.123.123");
blockList.addRange("10.0.0.1", "10.0.0.10");
blockList.addSubnet("8592:757c:efae:4e45::", 64, "ipv6");

console.log(blockList.check("123.123.123.123")); // Prints: true
console.log(blockList.check("10.0.0.3")); // Prints: true
console.log(blockList.check("222.111.111.222")); // Prints: false

// IPv6 notation for IPv4 addresses works:
console.log(blockList.check("::ffff:7b7b:7b7b", "ipv6")); // Prints: true
console.log(blockList.check("::ffff:123.123.123.123", "ipv6")); // Prints: true
```

Here's how I turned it into an Express middleware:

```js
// generate this however you need to
// I had a list of CIDR ranges like 198.51.100.14/24
const internalIps = [...];

// technically, this will be an allow list
const internal = new net.BlockList();

// add all to internal
internalIps.forEach((range) => {
    const [ip, mask] = range.split("/");

    internal.addSubnet(ip, +mask, "ipv4");
  });

module.exports = { internal, internalIps };

function checkIp(req, res, next) {
  // if it's not internal, send a 403
  if (!internal.check(req.ip)) {
    res.status(403).send("Not allowed");
  }

  // if we get here, send it through
  next();
}
```

And then this can be plugged into Express like this:

```js
app.use("/internal", checkIp);
```

And any path starting with `/internal` will check that the user's IP address matches our allowed range.
