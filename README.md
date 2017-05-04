# Samba JCIFS

The upstream project site is:
* https://jcifs.samba.org/

Versions notes:
* `1.3.17` - is the most recent Maven artifact available in the public repos
* `1.3.18` - is the most recent release from the Samba JCIFS project


## Establishing the Source Repository

This repository was built in the following manner:
* The source for `1.3.17` from the public maven repos
   * Reorganised into a Maven-like structure
   * Removed non-source/non-resource files
   * Added on the `jcifs-project` branch, tagged as `jcifs-1.3.17`
* The source for `1.3.18` from the project site (filename `jcifs-1.3.18.zip`)
   * Reorganised into a Maven-like structure
   * Removed non-source/non-resource files
   * Added on the `jcifs-project` branch, tagged as `jcifs-1.3.18`


## Local Changes

### Skip Negotiate Port Fallback

Added new configuration option `jcifs.skipNegotiatePortFallback` to allow us to
disable the default retry behaviour if the initial negotiate call fails.  By default the
following may happen:
* if port 0 or 445 (default port) retry using port 139
* otherwise retry using port 445

Typically this will _always_ fail and simply add another 30 seconds to the process for the
extra connection attempt to timeout.


### Multi-host Retry Workaround/Fix

In `jcifs.smb.SmbFile` when there are multiple addresses available for the
service (see the `getAddress()` implementation) the `doConnect()` method may be
called multiple times.

It appears that what should happen is the `doConnect()` method will get call for
each available address value until it either succeeds or all addresses have been
attempted.  The result of `getAddress()` gets moved on by a `getNextAddress()`
call.

What currently happens (in the 1.3.18 codebase) is when the first address fails
the `doConnect()` method will be called multiple times (equal to the number of
available addresses), but the first address will be used on every attempt.  This
is because the address is only set on transport creation, but if a `tree`
already exists it will be reused.

This becomes an issue when there are multiple hosts returned for a name, one of
them is down and that bad one is the first returned.  In an observed DFS share 
case there were 12 Active Directory hosts returned for the name.  Using the 
default JCIFS timeouts resulted in a failure after waiting about 6 minutes.

#### Reproducing the Issue

The easiest way to reproduce this issue (short of running a local DNS service
with a specifically crafted entry) is to
* patch the `RESOLVER_DNS` case in `jcifs.UniAddress#getAllByName` and have it
  insert a non-existant IP as the first item
* set the system property `-Djcifs.resolveOrder=DNS`

Something like:
```
case RESOLVER_DNS:
    if( isAllDigits( hostname )) {
        throw new UnknownHostException( hostname );
    }
    InetAddress[] iaddrs = InetAddress.getAllByName( hostname )
    UniAddress[] addrs = new UniAddress[iaddrs.length + 1];
	addrs[0] = new UniAddress(InetAddress.getByName("10.9.9.9"));  // bad first address
    for (int ii = 0; ii < iaddrs.length; ii++) {
        addrs[ii + 1] = new UniAddress(iaddrs[ii]);
    }
    return addrs; // Success
```
