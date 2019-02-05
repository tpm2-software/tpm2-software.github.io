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
