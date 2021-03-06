## ocaml-rfc6287
[![Build Status](https://travis-ci.org/sg2342/ocaml-rfc6287.svg?branch=master)](https://travis-ci.org/sg2342/ocaml-rfc6287)

OCaml implementation of [RFC6287](http://tools.ietf.org/html/rfc6287) OCRA (OATH Challenge-Response Algorithm)
incl. [RFC Errata ID: 3729](http://www.rfc-editor.org/errata_search.php?eid=3729)

## Notes on bisect and ounit tests

```
$ CONFIGUREFLAGS="--enable-tests --enable-coverage" make distclean cover_test
```

## Example usage

enable and build ocra_tool example
```
$ CONFIGUREFLAGS="--enable-examples" make distclean build
```

create credential files for client and server (server has a counter window of 5)
```
$ S=OCRA-1:HOTP-SHA1-6:C-QN08-PSHA1
$ K=0x00112233445566778899aabbccddeeff00112233
$ P=1234
$ C=0
$ ./ocra_tool.native init -f /tmp/client.ocra -s $S -k $K -p $P -c $C
$ ./ocra_tool.native info -f /tmp/client.ocra
suite:            OCRA-1:HOTP-SHA1-6:C-QN08-PSHA1
key:              0x00112233445566778899aabbccddeeff00112233
counter:          0x0
pinhash:          0x7110eda4d09e062aa5e4a390b0a572ac0d2c0220

$ ./ocra_tool.native init -f /tmp/server.ocra -s $S -k $K -p $P -c $C -w 5
$ ./ocra_tool.native info -f /tmp/server.ocra
suite:            OCRA-1:HOTP-SHA1-6:C-QN08-PSHA1
key:              0x00112233445566778899aabbccddeeff00112233
counter:          0x0
pinhash:          0x7110eda4d09e062aa5e4a390b0a572ac0d2c0220
counter_window:   5
```

generate challenge (server), calculate response (client) and
verify (server).

counter values in credential files have been incremented
```
$ Q=`./ocra_tool.native challenge -f /tmp/server.ocra`
$ A=`./ocra_tool.native response -f /tmp/client.ocra -q $Q`
$ ./ocra_tool.native verify -f /tmp/server.ocra -q $Q -a $A
success

$ ./ocra_tool.native info -f /tmp/server.ocra
suite:            OCRA-1:HOTP-SHA1-6:C-QN08-PSHA1
key:              0x00112233445566778899aabbccddeeff00112233
counter:          0x1
pinhash:          0x7110eda4d09e062aa5e4a390b0a572ac0d2c0220
counter_window:   5
$ ./ocra_tool.native info -f /tmp/client.ocra
suite:            OCRA-1:HOTP-SHA1-6:C-QN08-PSHA1
key:              0x00112233445566778899aabbccddeeff00112233
counter:          0x1
pinhash:          0x7110eda4d09e062aa5e4a390b0a572ac0d2c0220
```