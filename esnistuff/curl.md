
# Integrating curl

20190301

I'm starting to play with integrating this openssl build into curl.
I've not yet added any new stuff to curl, just building for now.

- curl can be built with openssl
- curl has some concept of using c-ares for DNS lookups, which we'll
  need to get ESNIKeys value(s)

I forked a version of curl for this from https://github.com/curl/curl

## curl Preliminaries

- clone curl:

            $ cd $HOME/code
            $ git clone https://github.com/sftcd/curl.git

- install c-ares (I did already for [wget](wget.md))

            $ sudo apt install libc-ares-dev

- get ready... and... build:

            $ cd $HOME/code/curl
            $ ./buildconf
            $  export LDFLAGS="-L$HOME/code/openssl"
            $ ./configure --with-ssl=$HOME/code/openssl --enable-ares
            $ make
            

    And that seems to work, I can at least run the resulting binary:

            $ export LD_LIBRARY_PATH=$HOME/code/openssl
            $ src/curl https://github.com/sftcd/openssl/blob/master/esnistuff/curl.md
            ... expected output, being this file:-) ...

## Coding 

All new ESNI code will be protected via ``#ifndef OPENSSL_NO_ESNI`` as
done within the OpenSSL library. 

Just as a first step, I added a ``#include <esni.h>`` to ``lib/vtls/openssl.c``
and all seems well.

Further exploration of methods already in use in __curl__ code
suggests using a special symbol, `USE_ESNI` to protect ESNI code
blocks.  This is now defined in *lib/curl_setup.h*, using the
following code.

```
/* Single point where USE_ESNI definition might be defined */
#if defined(USE_OPENSSL) && !defined(OPENSSL_NO_ESNI) && \
  defined(HAVE_OPENSSL_ESNI_H)
# define USE_ESNI               /* ESNI support available */
#else
# undef USE_ESNI                /* No ESNI support */
#endif
```

## Command line arguments, and behaviour

Most of this will be modelled on what we did for ``openssl s_client`` with
the difference that curl has the ``c-ares`` DNS libarary so can try
fetch ESNIKeys values internally.

Our model here is that if ESNI is available, we'll use it. However, given
that we may need to specify a specific cover hostname might be needed, 
and that ESNI will be initially rare, we'll add a command line argument
like ``--esni[=COVER]`` that can take an optional covername. 

