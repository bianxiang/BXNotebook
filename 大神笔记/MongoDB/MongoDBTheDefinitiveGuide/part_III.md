[toc]

## Part III. 复制

## （未）9. Setting Up a Replica Set

### 9.1 Introduction to Replication

Since the first chapter, we’ve been using a standalone server, a single mongod server. Replication is a way of keeping identical copies of your data on multiple servers and is recommended for all production deployments. 复制一方面可以保证数据安全，另一方面保证应用可用（一台出问题还有另外一台）。

With MongoDB, you set up replication by creating a **replica set**. A replica set is a group of servers with one primary, the server taking client requests, and multiple secondaries, servers that keep copies of the primary’s data. If the primary crashes, the secondaries can elect a new primary from amongst themselves.

### 9.2 A One-Minute Test Setup

This section will get you started quickly by setting up a three-member replica set on your local machine. This setup is obviously not suitable for production, but it’s a nice way to familiarize yourself with replication and play around with configuration.

Start up a mongo shell with the `--nodb` option, which allows you to start a shell that is not connected to any mongod:

	$ mongo --nodb

Create a replica set by running the following command:

	replicaSet = new ReplSetTest({"nodes" : 3})

However, it doesn’t actually start the mongod servers until you run the following two commands:

    > // starts three mongod processes
    > replicaSet.startSet()
    > // configures replication
    > replicaSet.initiate()

You should now have three mongod processes running locally on ports 31000, 31001, and 31002. They will all be dumping their logs into the current shell, which is very noisy, so put this shell aside and open up a new one.

In the second shell, connect to the mongod running on port 31000:

    > conn1 = new Mongo("localhost:31000")
    connection to localhost:31000
    testReplSet:PRIMARY>
    testReplSet:PRIMARY> primaryDB = conn1.getDB("test") test

Notice that, when you connect to a replica set member, the prompt changes to `testReplSet:PRIMARY>`. "PRIMARY" is the state of the member and "testReplSet" is an identifier for this set. You’ll learn how to choose your own identifier later; testRepl Set is the default name ReplSetTest uses.

Use your connection to the primary to run the isMaster command. This will show you the status of the set:
    > primaryDB.isMaster()
    {
        "setName" : "testReplSet",
        "ismaster" : true,
        "secondary" : false,
        "hosts" : [
            "wooster:31000",
            "wooster:31002",
            "wooster:31001"
        ],
        "primary" : "wooster:31000",
        "me" : "wooster:31000",
        "maxBsonObjectSize" : 16777216,
        "localTime" : ISODate("2012-09-28T15:48:11.025Z"),
        "ok" : 1
    }

Now that you’re connected to the primary, let’s try doing some writes and see what happens. First, insert 1,000 documents:

    > for (i=0; i<1000; i++) { primaryDB.coll.insert({count: i}) }
    > // make sure the docs are there
    > primaryDB.coll.count()
    1000

Now check one of the secondaries and verify that they have a copy of all of these docu‐ ments. Connect to either of the secondaries:

    > conn2 = new Mongo("localhost:31001")
    connection to localhost:31001
    > secondaryDB = conn2.getDB("test")
    test

Secondaries may fall behind the primary (or lag) and not have the most current writes, so secondaries will refuse read requests by default to prevent applications from accidentally reading stale data. Thus, if you attempt to query a secondary, you’ll get an error that it’s not primary:

	secondaryDB.coll.find()
    error: { "$err" : "not master and slaveok=false", "code" : 13435 }

This is to protect your application from accidentally connecting to a secondary and reading stale data. To allow queries on the secondary, we set an “I’m okay with reading from secondaries” flag, like so:

	> conn2.setSlaveOk()

Note that `slaveOk` is set on the **connection** (conn2), not the database (secondaryDB).

现在就可以正常读了。


You can see that all of our documents are there. Now, try to write to a secondary:

    > secondaryDB.coll.insert({"count" : 1001})
    > secondaryDB.runCommand({"getLastError" : 1})
    {
        "err" : "not master",
        "code" : 10058,
        "n" : 0,
        "lastOp" : Timestamp(0, 0),
        "connectionId" : 5,
    "ok" : 1 }

You can see that the secondary does not accept the write. The secondary will only perform writes that it gets through replication, not from clients.

There is one other interesting feature that you should try out: automatic failover. If the primary goes down, one of the secondaries will automatically be elected primary. To try this out, stop the primary:

	> primaryDB.adminCommand({"shutdown" : 1})

Run isMaster on the secondary to see who has become the new primary:

	> secondaryDB.isMaster()

It should look something like this:

    {
    	"setName" : "testReplSet",
        "ismaster" : true,
        "secondary" : false,
        "hosts" : [
            "wooster:31001",
            "wooster:31000",
            "wooster:31002"
        ],
        "primary" : "wooster:31001",
        "me" : "wooster:31001",
        "maxBsonObjectSize" : 16777216,
        "localTime" : ISODate("2012-09-28T16:52:07.975Z"),
        "ok" : 1
    }

Your primary may be the other server; whichever secondary noticed that the primary was down first will be elected. Now you can send writes to the new primary.

`isMaster` is a very old command, predating replica sets to when MongoDB only sup‐ ported master-slave replication. Thus, it does not use the replica set terminology consistently: it still calls the primary a “master.” You can generally think of “master” as equivalent to “primary” and “slave” as equivalent to “secondary.”

To shutdown the set, run:

	> replicaSet.stopSet()

一些关键概念：

- 客户端可以向主服务器发任何操作。
- 客户端不能向二级服务器写
- 客户端默认页不能读二级服务器。By explicitly setting an “I know I’m reading from a secondary” setting, clients can read from secondaries.

### 9.3 Configuring a Replica Set

现实中，复制要分布在多台机器上。

Let’s say that you already have a standalone mongod on server-1:27017 with some data on it. The first thing you need to do is choose a name for your set. Any string whatsoever will do, so long as it’s UTF-8.

Once you have a name for your replica set, restart server-1 with the `--replSet` name option. For example:

	$ mongod --replSet spock -f mongod.conf --fork

Now start up two more mongod servers with the replSet option and the same identifier (spock): these will be the other members of the set:

    $ ssh server-2
    server-2$ mongod --replSet spock -f mongod.conf --fork
    server-2$ exit
    $
    $ ssh server-3
    server-3$ mongod --replSet spock -f mongod.conf --fork
    server-3$ exit

For each member, add the replSet option to its mongod.conf file so that it will be used on startup from now on.

However, each mongod does not yet know that the others exist. To tell them about one another, you have to create a configuration that lists each of the members and send this configuration to server-1. It will take care of propagating it to the other members.

     config = {
        "_id" : "spock",
        "members" : [
            {"_id" : 0, "host" : "server-1:27017"},
            {"_id" : 1, "host" : "server-2:27017"},
            {"_id" : 2, "host" : "server-3:27017"}
		]
    }

The config’s "_id" is the name of the set that you passed in on the command line (in this example, "spock"). Make sure that this name matches exactly.

The next part of the document is an array of members of the set. Each of these needs two fields: a unique "_id" that is an integer and a hostname (replace the hostnames with whatever your servers are called).

This config object is your replica set configuration, so now you have to send it to a member of the set. To do so, connect to the server with data on it (server-1:27017) and initiate the set with this configuration:

    > // connect to server-1
    > db = (new Mongo("server-1:27017")).getDB("test") >
    > // initiate replica set
    > rs.initiate(config)
    {
    	"info" : "Config now saved locally.  Should come online in about a minute.",
        "ok" : 1
    }

server-1 will parse the configuration and send messages to the other members, alerting them of the new configuration. Once they have all loaded the configuration, 它们将选择一个作为主服务器，开始处理读写。

> Unfortunately, you cannot convert a standalone server to a replica set without some downtime for restarting it and initializing the set. Thus, even if you only have one server to start out with, you may want to configure it as a one-member replica set. That way, if you want to add more members later, you can do so without downtime.

If you are starting a brand-new set, you can send the configuration to any member in the set. If you are starting with data on one of the members, you must send the configuration to the member with data. You cannot initiate a set with data on more than one member.

> You must use the mongo shell to configure replica sets. There is no way to do file-based replica set configuration.

#### rs Helper Functions

Note the `rs` in the rs.initiate() command above. rs is a global variable that contains replication helper functions (run rs.help() to see the helpers it exposes). These functions are almost always just wrappers around database commands. For example, the following database command is equivalent to `rs.initiate(config)`:

	> db.adminCommand({"replSetInitiate" : config})

It is good to have a passing familiarity with both the helpers and the underlying commands, as it may sometimes be easier to use the command form instead of the helper.

#### Networking Considerations

Every member of a set must be able to make connections to every other member of the set (including itself). If you get errors about members not being able to reach other members that you know are running, you may have to change your network configuration to allow connections between them.

MongoDB allows all-localhost replica sets for testing locally but will protest if you try to mix localhost and non-localhost servers in a config.

### 9.4 Changing Your Replica Set Configuration

Replica set configurations can be changed at any time: members can be added, removed, or modified. There are shell helpers for some common operations; for example, to add a new member to the set, you can use rs.add:

	> rs.add("server-4:27017")

Similarly, you can remove members;

	> rs.remove("server-1:27017")
    Fri Sep 28 16:44:46 DBClientCursor::init call() failed
    Fri Sep 28 16:44:46 query failed : admin.$cmd { replSetReconfig: {
        _id: "testReplSet", version: 2, members: [ { _id: 0, host: "ubuntu:31000" },
       { _id: 2, host: "ubuntu:31002" } ] } } to: localhost:31000
    Fri Sep 28 16:44:46 Error: error doing query:
        failed src/mongo/shell/collection.js:155
    Fri Sep 28 16:44:46 trying reconnect to localhost:31000
    Fri Sep 28 16:44:46 reconnect localhost:31000 ok

Note that when you remove a member (or do almost any configuration change other than adding a member), you will get a big, ugly error about not being able to connect to the database in the shell. This is okay; it actually means the reconfiguration succeeded! When you reconfigure a set, the primary closes all connections as the last step in the reconfiguration process. Thus, the shell will briefly be disconnected but will automatically reconnect on your next operation.

The reason that the primary closes all connections is that it briefly steps down whenever you reconfigure the set. It should step up again immediately, but be aware that your set will not have a primary for a moment or two after reconfiguring.

You can check that a reconfiguration succeeded by run rs.config() in the shell. It will print the current configuration:
    > rs.config()
    {
        "_id" : "testReplSet",
        "version" : 2,
        "members" : [
            {
            	"_id" : 1,
                "host" : "server-2:27017"
            },
            {
            	"_id" : 2,
                "host" : "server-3:27017"
            },
            {
            	"_id" : 3,
                "host" : "server-4:27017"
			}
        ]
    }

Each time you change the configuration, the "version" field will increase. It starts at version 1.

You can also modify existing members, not just add and remove them. To make mod‐ ifications, create the configuration document that you want in the shell and call `rs.reconfig`.

Someone accidentally added member 1 by IP, instead of its hostname. To change that, first we load the current configuration in the shell and then we change the relevant fields:

    > var config = rs.config()
    > config.members[1].host = "server-2:27017"

Now that the config document is correct, we need to send it to the database using the rs.reconfig helper:

    > rs.reconfig(config)

rs.reconfig is often more useful that rs.add and rs.remove for complex operations, such as modifying members’ configuration or adding/removing multiple members at once. You can use it to make any legal configuration change you need: simply create the config document that represents your desired configuration and pass it to `rs.reconfig`.

### 9.5 How to Design a Set

To plan out your set, there are certain replica set concepts that you must be familiar with. The next chapter goes into more detail about these, but the most important is that replica sets are all about **majorities**: you need a majority of members to elect a primary, a primary can only stay primary so long as it can reach a majority, and a write is safe when it’s been replicated to a majority. This majority is defined to be “more than half of all members in the set,” as shown in Table 9-1.




