Chat based on Node.js using Redis Pub/Sub + socket.io
======================================================

A simple real-time chat service and jQuery plugin using redis Pub/Sub and socket.io.

Supports HMAC authentication, multiple chat rooms and chat history.

How to get the application running
------------------------------------

1. Install everything

 * <a href="http://redis.io">Redis</a> must be installed and running.
 * <a href="http://nodejs.org">Node.js</a> must be installed
 * Node.js plugin <a href="http://socket.io">socket.io</a> installed (npm install socket.io).
 * Node.js plugin <a href="https://github.com/mranney/node_redis">redis</a> installed (npm install hiredis redis).


2. Start the service via:

<code>$ node chatitat.js</code>


3. Start the demonstration client via:

<code>$ python -m SimpleHTTPServer</code>

in the client/ directory

Setting up the Client
-----------------------

The browser client is a jQuery plugin. It requires jQuery and Socket.io (the socket.io-client js can be used, or the version served by the service at /socket.io/socket.io.js)

The jQuery plugin can be attached to an element as follows:

    $('#chat').chatitat({
        // optional authentication information (used if a shared secret is supplied to the service)
        issued: "1333333333337", // timestamp at which the authentication was issued
        signature: "HASH", // a base64 digest of a SHA256 HMAC using
                           // the server's secret salt and userID + '|' + channel + '|' + issued
        
        // connection
        userID: 'fred.flintstone',
        userName: 'Fred Flintstone',
        host: 'http://localhost:8020',
        channel: 'bedrock',
        
        // callbacks, for convenience
        errorCallback: function(msg, user, name, channel) {
            console.log('Error', msg, user, name, channel);
        },
        sendCallback: function(msg, user, name, channel) {
            console.log('Sending', msg, user, name, channel);
        },
        receiveCallback: function(msg, user, name, channel) {
            console.log('Receiving', msg, user, name, channel);
        }
    });


Authentication
---------------

The chat service uses a shared secret between the server hosting the browser client page and the chat service. This shared secret is never exposed to the client and is used as a salt to hash the user id, channel and issued timestamp using a SHA256 HMAC.

This hash signature is encoded as base64 and sent to the client, which can use this hash to prove the user id and channel are indeed those issued by the server (and not forgeries). The hash expires after the sessionLength specified in the service's settings.js file. This is policed by sending the time of issue as a unix timestamp (which is also part of the signature hash).

The shared secret is defined in settings.js:

e.g.

    module.exports = {
        secret: 'some long secret shared key for authenticating sessions',
        sessionLength: 60*60*24, // 24h time out
        port: 8020,
        messageId: 'chat-id', // incrementing unique id for messages
        messageHash: 'chat-message', // prefix for redis chat message store
        history: 'chat-history', // prefix for redis channel history
        subscription: 'chat' // prefix for redis channel subscription
    };


REST API
--------

The chat service also has a REST API. This can be used for:
* Creating an HMAC from user, channel and issued stamp
* Retrieving a channel's chat history for archival
* Deleting part of a channel's chat history which has been archived (to free memory)


#### HMAC
The path:
<code>/hmac/salt/user_id/channel/issued_timestamp/</code>
will generate a base64 HMAC signature for a given salt (secret), given the user, channel and issued timestamp

This should be used over HTTPS, or as a reference implementation to ensure the secret salt is not stolen.
The signature is generated by combining the <code>user_id + '|' + channel + '|' + issued_timestamp</code> together (Note: | is used as a separator), which is passed through a SHA256 HMAC. The digest is read as base64.

#### History
The path:
<code>/history/channel</code>
will list the entire chat history buffer for the given channel as JSON.

<code>/history/channel/10</code>
will list the 10 oldest history entries in the channel history buffer.

----

This requires the same HMAC authentication via query string. e.g.
<code>/history/channel?user=fred.flintstone&issued=1333333333337&signature=HMAC+HASH</code>

<b>Note:</b> If a DELETE instead of a GET request is used, the history will be deleted.
<code>/history/channel/10</code>
will delete the 10 oldest history entries in the buffer.

FAQ
----

#### Why is it called Chatitat?
I don't know. Can you think of a better name?

#### Why aren't there more FAQs?
No-one has asked any questions yet.


# Credits
This was loosely based off <a href="https://github.com/steffenwt/nodejs-pub-sub-chat-demo">nodejs-pub-sub-chat-demo</a>.
