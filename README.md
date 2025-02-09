# swift_pro_telegram_bot

## configuration to scale

.env file contains environment of node application.\
You should add "SCALING" to .env file to enable scaling.\
SCALING='left' means first instance of scaling\
SCALING='right' means second instance of scaling\
### Example of .env file of first scaling instance
```
...
MAX_CORES='yes'
SCALING='left'
```

### Example of .env file of second scaling instance
```
...
MAX_CORES='yes'
SCALING='right'
```

## node modules to fix

### web3-core-helpers
```
node_modules/web3-core-helpers/lib/formatters.js
node_modules/web3-core-helpers/src/formatters.js

in function outputTransactionFormatter

    tx.gas = utils.hexToNumber(tx.gas);
to
    if (tx.gas) tx.gas = outputBigNumberFormatter(tx.gas);
```

### web3-providers-http

```
node_modules/web3-providers-http/lib/index.js
node_modules/web3-providers-http/src/index.js

in HttpProvider.prototype.send

    // Response is a stream data so should be awaited for json response
    response.json().then(function (data) {
        callback(null, data);
    }).catch(function (error) {
        callback(errors.InvalidResponse(response));
    });
to
    const h = this.host
    // Response is a stream data so should be awaited for json response
    response.json().then(function (data) {
        callback(null, data);
    }).catch(function (error) {
        // callback(errors.InvalidResponse(response));
        console.log('###', h, payload[0]? payload[0]: payload, error)
        callback(null, { /// web3-core-requestmanager/lib/jsonrpc.js:65 - Jsonrpc.isValidResponse
            jsonrpc: '2.0',
            id: 0,
            result: {}
        });
    });
```

Please set node option to allocate 64GB RAM
```
export NODE_OPTIONS="--max-old-space-size=65536"
```

## How to run/stop the bot

You can login to the bash shell of machines by ssh
You run the following commands in order

```
pm2 stop 0
cd /home/ubuntu/app
NODE_ENV=production pm2 start build/index.js --node-args="--max-old-space-size=49152" -f
```

You can check if the bot processes are running
```
pm2 list
```

You can see the current running logs
```
pm2 log 0
```

## How to purge messages

Bot might be stopped for any reason.\
Some messages are pushed to fifo message queue. You need to purge those messages before executing the bot.

You connect to the db by any means, including shell, client tool, VSCode extension, etc.\
Please update the field "purgeMessages" to "true" before executing the bot.

For example, the command in mongosh is the following.
```
db.apps.updateMany({},{$set:{purgeMessages:true}})
```

Then execute the bot and wait until all messages are purged.\
And restore the field to "false".
```
db.apps.updateMany({},{$set:{purgeMessages:false}})
```
After setting to "false", the bot will handle all messages from fifo message queue.