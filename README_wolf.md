# OpenVSwitch with wolfSSL

Below describes the steps for building OpenVSwitch with wolfSSL.

## wolfSSL

Support for OpenVSwitch added in: https://github.com/wolfSSL/wolfssl/pull/2399

```sh
git clone https://github.com/wolfSSL/wolfssl.git
cd wolfssl
./autogen.sh
./configure --enable-opensslall --enable-keygen --enable-rsapss --enable-aesccm \
    --enable-aesctr --enable-des3 --enable-camellia --enable-curve25519 --enable-ed25519 \
    --enable-sessioncerts \
    CFLAGS="-DWOLFSSL_LOG_PRINTF -DWOLFSSL_PUBLIC_MP -DWOLFSSL_DES_ECB"
make
make check
sudo make install
```

## Strongswan

```sh
sudo apt-get install flex bison byacc libsoup2.4-dev gperf

git clone https://github.com/strongswan/strongswan.git
cd strongswan
./autogen.sh #if packages are missing autogen.sh must be re-run
./configure --disable-defaults --enable-pki --enable-wolfssl --enable-pem
make
make check
sudo make install
```

If getting `proposal_keywords_static` error run:

```
cd src/libstrongswan
sed \
    -e "s:\@GPERF_LEN_TYPE\@:unsigned:" \
    crypto/proposal/proposal_keywords_static.h.in > crypto/proposal/proposal_keywords_static.h
cd ../..
```

### Strongswan Test Results

```
make check
...
Passed all 34 'libstrongswan' suites
PASS: libstrongswan_tests
=============
1 test passed
=============
```


## OpenVSwitch (OVS)

```sh
git clone https://github.com/openvswitch/ovs.git
cd ovs

./boot.sh
./configure --with-wolfssl
make
sudo make install
sudo make check
```

Note: Contribution PR for OVS with wolfSSL is here: https://github.com/dgarske/ovs/tree/wolf

### OVS Test Results

Test instructions:
http://docs.openvswitch.org/en/latest/topics/testing/

```
sudo make check

## ------------------------------- ##
## openvswitch 2.12.90 test suite. ##
## ------------------------------- ##

appctl bashcomp unit tests

  1: appctl-bashcomp - basic verification            ok
  2: appctl-bashcomp - complex completion check 1    ok
...

## ------------- ##
## Test results. ##
## ------------- ##

ERROR: 2449 tests were run,
39 failed unexpectedly.
342 tests were skipped.
## -------------------------- ##
## testsuite.log was created. ##
## -------------------------- ##

Please send `tests/testsuite.log' and all information you think might help:

   To: <bugs@openvswitch.org>
   Subject: [openvswitch 2.12.90] testsuite: 542 543 544 545 546 547 1921 1927 1928 1929 1930 1931 1932 1933 1934 1935 1936 1937 1938 1939 1940 1941 1942 1943 1944 1945 1946 1947 1948 1949 1950 1951 1952 2456 2500 2501 2658 2659 2660 failed
```

#### Debugging Tests

Running a single test:

```sh
OVS_PAUSE_TEST=1 make check TESTSUITEFLAGS='-v 542'
```

Debugging a single test:
In `./tests/testsuite.dir/[testnum]/run` find command used.

```sh
cd tests
sudo ./testsuite -v -d  AUTOTEST_PATH='utilities:vswitchd:ovsdb:vtep:tests:::ovn/controller-vtep:ovn/northd:ovn/utilities:ovn/controller' 1
```

Enabling debug output to console:

```sh
sudo ovs-appctl vlog/list
sudo ovs-appctl vlog/set ANY:console:dbg
```


## Support

For questions or issue please email support@wolfssl.com
