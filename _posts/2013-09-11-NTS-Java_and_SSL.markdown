---
layout: post
title: (NTS) Java and SSL
---

If you try to open SSL connections to a site that uses a untrusted certificate, like a self signed certificate used for development purposes or a certificate that is not in the "standard" certificate lists then you might incur in a `javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException` complaining that `PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target`

You have two ways to work around this problem: one is by modifying your code if you can do so, the other is to add the certificate to your JRE keystore.

The second one is less invasive and it consists of the following steps:

* Save the site certificate on your machine. You can do this using your browser.
* Use `keytool` to import the certicate into your JRE keystore:

        $ keytool -importcert -alias "Your description" -file certificate_file -keystore $JAVA_HOME/jre/lib/security/jssecacerts

Set a password for your keystore, answer `yes` to the question whether the certificate should be trusted and that's it.

You can re-run your code and everything should work fine.

