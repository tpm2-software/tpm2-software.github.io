---
layout: post
title:  "Getting Started"
date:   2019-02-05 18:28:05 +0100
categories: tpm2-tss getting-started
tags: bash make code
---

This tutorial is pretty much just a placeholder to show how future tutorials can be created here inside jekyll.

Stay put for future updates.

{% highlight bash %}
git clone https://github.com/tpm2-software/tpm2-tss
cd tpm2-tss
./bootstrap
./configure --enable-integration
make -j$(nproc)
make check
{% endhighlight %}

Check out the [tpm2-tss README] and [tpm2-tss INSTALL] for more information.

[tpm2-tss README]: https://github.com/tpm2-software/tpm2-tss/blob/master/README.md
[tpm2-tss INSTALL]: https://github.com/tpm2-software/tpm2-tss/blob/master/INSTALL.md


After successful installation a standalone application using ESAPI can be compiled as follows:

{% highlight bash %}
gcc standalone-example.c -L=/usr/local/lib/ -ltss2-esys -o standalone-example
{% endhighlight %}

Simple example standalone application:

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>
#include <tss2/tss2_esys.h>

int main() {

    TSS2_RC r;

    /* Initialize the ESAPI context */
    ESYS_CONTEXT *ctx;
    r = Esys_Initialize(&ctx, NULL, NULL);

        if (r != TSS2_RC_SUCCESS){
        printf("\nError: Esys_Initializen\n");
        exit(1);
    }

    /* Get random data */
    TPM2B_DIGEST *random_bytes;
    r = Esys_GetRandom(ctx, ESYS_TR_NONE, ESYS_TR_NONE, ESYS_TR_NONE, 20,
                       &random_bytes);

    if (r != TSS2_RC_SUCCESS){
        printf("\nError: Esys_GetRandom\n");
        exit(1);
    }

    printf("\n");
    for (int i = 0; i < random_bytes->size; i++) {
        printf("0x%x ", random_bytes->buffer[i]);
    }
    printf("\n");
    exit(0);
}
{% endhighlight %}
