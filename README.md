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
