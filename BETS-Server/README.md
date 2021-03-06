# Benchmarking E-ticketing systems #

This repository contains the Java code which can be run on a PC to operate the ACR122 NFC reader for the evaluation experiments on cryptographic protocols used during e-ticket issuing and usage.
The performance of these e-tickets protocols are evaluate with at least one party using an Android app which will communicate with an NFC reader to exchange tickets. This usually happens during the issuing stage or verification stage but other stages might involve NFC communication, too.

Currently, the code only works with a cheap (~£40) ACR122 NFC reader which is readily available from various online shops. The application has been mainly developed on Linux but as it is JAVA should work on other platforms with no or little change. 

## Setup ##

* Make sure you have a working ACR122 NFC reader attached to your PC
* Download and install the appropriate ACR122 drivers
* Test that the reader works using, for example, "pcsctest" or "pcsc_scan"
* Download this repository
* Import the repository into Eclipse as an "Existing Maven Project" (sometimes you may need to remove the files/diretories .settings, .classpath and .project)
* Run the program via the "uk.ac.surrey.bets_framework.Main" class, making sure that the "log" directory in the repository is added to the classpath as this contains the Logback properties file
* Output from the program is logged to "log/development.log"

## Command Options ##

To see all available options, use the "-h" or "--help" command line option.

### Logging ###

To enable logging, use "--log-level" ("-l").  This will enable logging on both the PC and the Android device at the specified level.  The default is to log everything.  Log output will be to both standard output/Android logcat and to the log/development.log file.

### Cryptography ###

Set the key length using "--key-length" ("-k").  The default is 1024.

Since the E-Ticket protocol uses a DH parameter set, this is automatically generated for the specified key length.  Since generation can be slow, use the "--output-dh" ("-o") to save the parameters to a file, then "--input-dh" ("-i") to load them in, bypassing generation.

### Running ###

To run a protocol, use the "--run" ("-r") option.  The name of the protocol must be specified.  Optionally, the number of times the protocol should be run can be specified, together with any parameters which are passed directly to the protocol.  For example:

--run Basic:10

will run the protocol with class name "Basic" 10 times without any further parameters.  Whereas:

-- run ETicket:1:10

will run the ETicket protocol once, with the protocol parameter "10".  Each parameter is separated by a ":".

To record the timings from the PC (server) and Android (client), use the following options:

* "--server-output" ("-s"): optionally output the server protocol timings to a named CSV file
* "--client-output" ("-c"): optionally output the client protocol timings to a named CSV file
* "--setup-output" ("-u"): optionally output the server setup timings to a named CSV file
* "--tear-down-output" ("-d"): optionally output the server tear down timings to a named CSV file

Each CSV file will have the following columns:

* Iteration number, starting at 0
* <Protocol>-Time: total number of milliseconds spent running the protocol on the client or server
* <Protocol>-Count: total number of times the protocol was run (on the server this will match the iterations, on the client this will match the number of times a protocol state was called)

And for each protocol state:

* <State>-Action-Time: total amount of time performing the required state action in milliseconds
* <State>-Action-Count: total number of times the state's action was called
* <State>-Command-Time: total amount of time performing the required state command (NFC communications) in milliseconds
* <State>-Command-Count: total number of times the state's command was called

You can add your own timing blocks in code with an associated name which will then each be associated with a "-Time" and "-Count" column.

## E-Ticket Protocol ##

The Guasch (2013) protocol has been implemented in the state machine class "ETicket"

The parameters available for the ETicket protocol are:

* (int) number of times ticket is to be used, e.g. 1 (default).
* (int) cost of each service, e.g. 1 (default) or 2, ... to have each iteration cost more to cause ticket verification failure.
* (String) encryption parameters, e.g. "RSA" (default) or "RSA/ECB/OAEPWithSHA1AndMGF1Padding"
* (String) hash parameters, e.g. "SHA1" (default) or "SHA16"
* (int) prime certainty, e.g. 80% (default) or above.

For example:

-r ETicket:1:2 -l 3 -k 512 -i log/dh_parameters.json -s log/server.csv -c log/client.csv -u log/setup.csv -d log/tear_down.csv

will run the ETicket protocol once, using each ticket twice, with log level 3 (info), key length 512, loading DH parameters from file, and saving server, client setup (server only) and tear down (server only) to files.

## PPETS-ABC Protocol ##

The Han et al (draft) protocol has been implemented in the state machine class "PPETSABC".

The parameters available for the PPETSABC protocol are:
* (boolean)skip verification tests which defaults to false.
* (int) the number of times that a ticket should be validated to provoke double spend, e.g. 2 (default).
* A,A1 or E is the type of pairing to use (see http://gas.dia.unisa.it/projects/jpbc/docs/ecpg.html)
* (int) number of r bits to use in Type A/E elliptic curve, e.g. 160 (default), for Type A1 this is the number of primes to use
* (int) number of q bits to use in Type A/E elliptic curve, e.g. 512 (default), for Type A1 this is the size of those primes

For example:

-r PPETSABC:1:false:2:E:512:1024 -l 3 -s log/server.csv -c log/client.csv -u log/setup.csv -d log/tear_down.csv

will run the PPETSABC protocol once, failing whenever a verification test fails, running ticket validation twice, with type E elliptic curve pairing where the prime r has bit length 512,  and the prime q has bit length 1024, with log level 3 (info), and saving server, client setup (server only) and tear down (server only) to files.

The skip verification option determines whether all the protocol verification steps continue even if a verification fails. Set this to true to ensure that the protocol continues to the end regardless of the verification outcome, or false to allow the protocol to fail at the relevant stage. 


## PPETS-ABCLite Protocol ##

There is also a "Lite" version of the PPETSABC protocol which has been implemented in the state machine class "PPETSABCLite".
The lite version has a simplified verification process which is less computationally expensive but as a consequence has slightly different security properties.

The parameters available for the PPETSABCLite protocol are the same as for the PPETSABC protocol, ie
* (boolean) skip verification tests which defaults to false.
* (int) the number of times that a ticket should be validated to provoke double spend, e.g. 2 (default).
* A,A1 or E is the type of pairing to use (see http://gas.dia.unisa.it/projects/jpbc/docs/ecpg.html)
* (int) number of r bits to use in Type A/E elliptic curve, e.g. 160 (default), for Type A1 this is the number of primes to use
* (int) number of q bits to use in Type A/E elliptic curve, e.g. 512 (default), for Type A1 this is the size of those primes

For example:

-r PPETSABCLite:1:false:2:A1 -l 4 -s log/server.csv -c log/client.csv -u log/setup.csv -d log/tear_down.csv

will run the PPETSABCLite protocol once, failing whenever a verification test fails, running ticket validation twice, with a type A1 elliptic curve pairing with the default 3 primes of size 160 bits and a log level 3 (info), as well as saving server, client setup (server only) and tear down (server only) to files.

The skip verification option determines whether all the protocol verification steps continue even if a verification fails. Set this to true to ensure that the protocol continues to the end regardless of the verification outcome, or false to allow the protocol to fail at the relevant stage. 

## AnonSSO Protocol ##

Han et al have proposed an "Anonymous Single-Sign-On for n designated services with traceability" (AnonSSO) protocol which has been implemented in the state machine class "AnonSSO".

The parameters available for the AnonSSO protocol are:
* (int) number of r bits to use in Type A elliptic curve, e.g. 160 (default).
* 0/1 validateVerifiers flag to indicate whether the Android client should validate the ticket details (0=yes, 1=no) setting this to 1 (ie do not validate the ticket details) will speed up the protocol run but defeats the purpose of the protocol and should only be done for testing purposes.



For example:

-r AnonSSO:1:320:0 -s log/serverAnonSSO_1.csv -c log/clientAnonSSO_1.csv -l 4

will run the AnonSSO protocol once with an elliptic curve whose elements are r=320 bit long, with log level 4 (debug), and only saving the server and client protocol timings to file.


## References ##

Guasch, A.V. (2013). "Contributions to the Security and Privacy of Electronic Ticketing Systems". Ph.D Dissertation, Universitat Rovira i Virgili.

Han, J., Chen, L., Schneider, S., Treharne, H., Wesemeyer, S. (draft). "PPETS-ABC: Privacy-preserving Electronic Ticket Scheme with Attribute-based Credentials".

Han, J., Chen, L., Schneider, S., Treharne, H., Wesemeyer, S. (draft). "Anonymous Single-Sign-On for n designated services with traceability" (http://arxiv.org/abs/1804.07201).
