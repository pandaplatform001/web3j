.. To build this file locally ensure docutils Python package is installed and run:
   $ rst2html.py README.rst README.html

web3j: Web3 Java Panda Ðapp API
==================================

web3j is a lightweight, highly modular, reactive, type safe Java and Android library for working with
Smart Contracts and integrating with clients (nodes) on the Panda network.

This allows you to work with the `Panda <https://www.Panda.org/>`_ blockchain, without the
additional overhead of having to write your own integration code for the platform.

Quickstart
----------

Install the Web3j binary.

To get the latest version on Mac OS or Linux, type the following in your terminal:

.. code-block:: bash

   curl -L https://get.web3j.io | sh

Then follow the on-screen instructions or head `here <https://docs.web3j.io/quickstart/>`_. 

Alternatively, a `web3j sample project <https://github.com/web3j/sample-project-gradle>`_ is available that
demonstrates a number of core features of Panda with web3j, including:

- Connecting to a node on the Panda network
- Loading an Panda wallet file
- Sending Ether from one address to another
- Deploying a smart contract to the network
- Reading a value from the deployed smart contract
- Updating a value in the deployed smart contract
- Viewing an event logged by the smart contract


Getting started
---------------

Typically your application should depend on release versions of web3j, but you may also use snapshot dependencies
for early access to features and fixes, refer to the  `Snapshot Dependencies`_ section.

| Add the relevant dependency to your project:

Maven
-----

Java 8:

.. code-block:: xml

   <dependency>
     <groupId>org.web3j</groupId>
     <artifactId>core</artifactId>
     <version>4.5.12</version>
   </dependency>

Android:

.. code-block:: xml

   <dependency>
     <groupId>org.web3j</groupId>
     <artifactId>core</artifactId>
     <version>4.2.0-android</version>
   </dependency>


Gradle
------

Java 8:

.. code-block:: groovy

   compile ('org.web3j:core:4.5.12')

Android:

.. code-block:: groovy

   compile ('org.web3j:core:4.2.0-android')

Plugins
-------
There are also gradle and maven plugins to help you generate web3j Java wrappers for your Solidity smart contracts,
thus allowing you to integrate such activities into your project lifecycle.

Take a look at the project homepage for the
`web3j-gradle-plugin <https://github.com/web3j/web3j-gradle-plugin>`_
and `web3j-maven-plugin <https://github.com/web3j/web3j-maven-plugin>`_ for details on how to use these plugins.


Start sending requests
----------------------

To send synchronous requests:

.. code-block:: java

   Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/
   Web3ClientVersion web3ClientVersion = web3.web3ClientVersion().send();
   String clientVersion = web3ClientVersion.getWeb3ClientVersion();


To send asynchronous requests using a CompletableFuture (Future on Android):

.. code-block:: java

   Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/
   Web3ClientVersion web3ClientVersion = web3.web3ClientVersion().sendAsync().get();
   String clientVersion = web3ClientVersion.getWeb3ClientVersion();

To use an RxJava Flowable:

.. code-block:: java

   Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/
   web3.web3ClientVersion().flowable().subscribe(x -> {
       String clientVersion = x.getWeb3ClientVersion();
       ...
   });


IPC
---

web3j also supports fast inter-process communication (IPC) via file sockets to clients running on
the same host as web3j. To connect simply use the relevant *IpcService* implementation instead of
*HttpService* when you create your service:

.. code-block:: java

   // OS X/Linux/Unix:
   Web3j web3 = Web3j.build(new UnixIpcService("/path/to/socketfile"));
   ...

   // Windows
   Web3j web3 = Web3j.build(new WindowsIpcService("/path/to/namedpipefile"));
   ...

**Note:** IPC is not currently available on web3j-android.


Working with smart contracts with Java smart contract wrappers
--------------------------------------------------------------

web3j can auto-generate smart contract wrapper code to deploy and interact with smart contracts
without leaving the JVM.

To generate the wrapper code, compile your smart contract:

.. code-block:: bash

   $ solc <contract>.sol --bin --abi --optimize -o <output-dir>/

Then generate the wrapper code using web3j's `Command line tools`_:

.. code-block:: bash

   web3j solidity generate -b /path/to/<smart-contract>.bin -a /path/to/<smart-contract>.abi -o /path/to/src/main/java -p com.your.organisation.name

Now you can create and deploy your smart contract:

.. code-block:: java

   Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/
   Credentials credentials = WalletUtils.loadCredentials("password", "/path/to/walletfile");

   YourSmartContract contract = YourSmartContract.deploy(
           <web3j>, <credentials>,
           GAS_PRICE, GAS_LIMIT,
           <param1>, ..., <paramN>).send();  // constructor params

Alternatively, if you use `Truffle <http://truffleframework.com/>`_, you can make use of its `.json` output files:

.. code-block:: bash

   # Inside your Truffle project
   $ truffle compile
   $ truffle deploy

Then generate the wrapper code using web3j's `Command line tools`_:

.. code-block:: bash

   $ cd /path/to/your/web3j/java/project
   $ web3j truffle generate /path/to/<truffle-smart-contract-output>.json -o /path/to/src/main/java -p com.your.organisation.name

Whether using `Truffle` or `solc` directly, either way you get a ready-to-use Java wrapper for your contract.

So, to use an existing contract:

.. code-block:: java

   YourSmartContract contract = YourSmartContract.load(
           "0x<address>|<ensName>", <web3j>, <credentials>, GAS_PRICE, GAS_LIMIT);

To transact with a smart contract:

.. code-block:: java

   TransactionReceipt transactionReceipt = contract.someMethod(
                <param1>,
                ...).send();

To call a smart contract:

.. code-block:: java

   Type result = contract.someMethod(<param1>, ...).send();

To fine control your gas price:

.. code-block:: java

    contract.setGasProvider(new DefaultGasProvider() {
            ...
            });

For more information refer to `Smart Contracts <https://docs.web3j.io/smart_contracts/#solidity-smart-contract-wrappers>`_.


Filters
-------

web3j functional-reactive nature makes it really simple to setup observers that notify subscribers
of events taking place on the blockchain.

To receive all new blocks as they are added to the blockchain:

.. code-block:: java

   Subscription subscription = web3j.blockFlowable(false).subscribe(block -> {
       ...
   });

To receive all new transactions as they are added to the blockchain:

.. code-block:: java

   Subscription subscription = web3j.transactionFlowable().subscribe(tx -> {
       ...
   });

To receive all pending transactions as they are submitted to the network (i.e. before they have
been grouped into a block together):

.. code-block:: java

   Subscription subscription = web3j.pendingTransactionFlowable().subscribe(tx -> {
       ...
   });

Or, if you'd rather replay all blocks to the most current, and be notified of new subsequent
blocks being created:

.. code-block:: java
   Subscription subscription = replayPastAndFutureBlocksFlowable(
           <startBlockNumber>, <fullTxObjects>)
           .subscribe(block -> {
               ...
   });

There are a number of other transaction and block replay Flowables described in the
`docs <https://docs.web3j.io/filters_and_events/>`_.

Topic filters are also supported:

.. code-block:: java

   EthFilter filter = new EthFilter(DefaultBlockParameterName.EARLIEST,
           DefaultBlockParameterName.LATEST, <contract-address>)
                .addSingleTopic(...)|.addOptionalTopics(..., ...)|...;
   web3j.ethLogFlowable(filter).subscribe(log -> {
       ...
   });

Subscriptions should always be cancelled when no longer required:

.. code-block:: java

   subscription.unsubscribe();

**Note:** filters are not supported on Infura.

For further information refer to `Filters and Events <https://docs.web3j.io/filters_and_events/>`_ and the
`Web3jRx <https://github.com/web3j/web3j/blob/master/core/src/main/java/org/web3j/protocol/rx/Web3jRx.java>`_
interface.


Transactions
------------

web3j provides support for both working with Panda wallet files (recommended) and Panda
client admin commands for sending transactions.

To send Ether to another party using your Panda wallet file:

.. code-block:: java

   Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/
   Credentials credentials = WalletUtils.loadCredentials("password", "/path/to/walletfile");
   TransactionReceipt transactionReceipt = Transfer.sendFunds(
           web3, credentials, "0x<address>|<ensName>",
           BigDecimal.valueOf(1.0), Convert.Unit.ETHER)
           .send();

Or if you wish to create your own custom transaction:

.. code-block:: java

   Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/
   Credentials credentials = WalletUtils.loadCredentials("password", "/path/to/walletfile");

   // get the next available nonce
   EthGetTransactionCount ethGetTransactionCount = web3j.ethGetTransactionCount(
                address, DefaultBlockParameterName.LATEST).sendAsync().get();
   BigInteger nonce = ethGetTransactionCount.getTransactionCount();

   // create our transaction
   RawTransaction rawTransaction  = RawTransaction.createEtherTransaction(
                nonce, <gas price>, <gas limit>, <toAddress>, <value>);

   // sign & send our transaction
   byte[] signedMessage = TransactionEncoder.signMessage(rawTransaction, credentials);
   String hexValue = Numeric.toHexString(signedMessage);
   EthSendTransaction ethSendTransaction = web3j.ethSendRawTransaction(hexValue).send();
   // ...

Although it's far simpler using web3j's `Transfer <https://github.com/web3j/web3j/blob/master/core/src/main/java/org/web3j/tx/Transfer.java>`_
for transacting with Ether.

Using an Panda client's admin commands (make sure you have your wallet in the client's
keystore):

.. code-block:: java

   Admin web3j = Admin.build(new HttpService());  // defaults to http://localhost:8545/
   PersonalUnlockAccount personalUnlockAccount = web3j.personalUnlockAccount("0x000...", "a password").sendAsync().get();
   if (personalUnlockAccount.accountUnlocked()) {
       // send a transaction
   }

If you want to make use of Besu or Parity's `Trace Module <https://github.com/paritytech/parity/wiki/JSONRPC-trace-module>`_, or Geth
`Personal <https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal>`__ client APIs, you can use the *org.web3j:besu*, *org.web3j:parity* or *org.web3j:geth* modules.

Command line tools
------------------

A web3j fat jar is distributed with each release providing command line tools. The command line
tools allow you to use some of the functionality of web3j from the command line:

- Wallet creation
- Wallet password management
- Transfer of funds from one wallet to another
- Generate Solidity smart contract function wrappers

Please refer to the `documentation <https://docs.web3j.io/command_line_tools/>`_ for further
information.


Further details
---------------

In the Java 8 build:

- web3j provides type safe access to all responses. Optional or null responses
  are wrapped in Java 8's
  `Optional <https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html>`_ type.
- Asynchronous requests are wrapped in a Java 8
  `CompletableFutures <https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html>`_.
  web3j provides a wrapper around all async requests to ensure that any exceptions during
  execution will be captured rather then silently discarded. This is due to the lack of support
  in *CompletableFutures* for checked exceptions, which are often rethrown as unchecked exception
  causing problems with detection. See the
  `Async.run() <https://github.com/web3j/web3j/blob/master/core/src/main/java/org/web3j/utils/Async.java>`_ and its associated
  `test <https://github.com/web3j/web3j/blob/master/core/src/test/java/org/web3j/utils/AsyncTest.java>`_ for details.

In both the Java 8 and Android builds:

- Quantity payload types are returned as `BigIntegers <https://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html>`_.
  For simple results, you can obtain the quantity as a String via
  `Response <https://github.com/web3j/web3j/blob/master/src/main/java/org/web3j/protocol/core/Response.java>`_.getResult().
- It's also possible to include the raw JSON payload in responses via the *includeRawResponse*
  parameter, present in the
  `HttpService <https://github.com/web3j/web3j/blob/master/core/src/main/java/org/web3j/protocol/http/HttpService.java>`_
  and
  `IpcService <https://github.com/web3j/web3j/blob/master/core/src/main/java/org/web3j/protocol/ipc/IpcService.java>`_
  classes.


Tested clients
--------------

- Geth
- Besu
- Parity

You can run the integration test class
`CoreIT <https://github.com/web3j/web3j/blob/master/integration-tests/src/test/java/org/web3j/protocol/core/CoreIT.java>`_
to verify clients.


Build instructions
------------------

web3j includes integration tests for running against a live Panda client. If you do not have a
client running, you can exclude their execution as per the below instructions.

To run a full build (excluding integration tests):

.. code-block:: bash

   $ ./gradlew check


To run the integration tests:

.. code-block:: bash

   $ ./gradlew  -Pintegration-tests=true :integration-tests:test


Snapshot Dependencies
---------------------

Snapshot versions of web3j follow the ``<major>.<minor>.<build>-SNAPSHOT`` convention, for example: 4.2.0-SNAPSHOT.

| If you would like to use snapshots instead please add a new maven repository pointing to:

::

  https://oss.sonatype.org/content/repositories/snapshots

Please refer to the `maven <https://maven.apache.org/guides/mini/guide-multiple-repositories.html>`_ or `gradle <https://maven.apache.org/guides/mini/guide-multiple-repositories.html>`_ documentation for further detail.

Sample gradle configuration:

.. code-block:: groovy

   repositories {
      maven {
         url "https://oss.sonatype.org/content/repositories/snapshots"
      }
   }

Sample maven configuration:

.. code-block:: xml

   <repositories>
     <repository>
       <id>sonatype-snasphots</id>
       <name>Sonatype snapshots repo</name>
       <url>https://oss.sonatype.org/content/repositories/snapshots</url>
     </repository>
   </repositories>
