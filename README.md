How to TLS
==========

I hear it a lot...how do I ignore certificate errors so my code will work?  Not aware of the implications of this, developers do it all the time.  Unfortunately, code samples are abundant for disabling TLS validation errors, when the easier thing to do - import certificates to trust (read: you don't have to write any code) is not all that well documented.  With that in mind, I'll gather some information here as a quick reference for developers instead of instructing their applications to ignore the errors.

* [Background](background.md)
    - Chain of Trust
    - CA certificates vs. self-signed
    - Certificate common name and DNS
* Importing to Windows, Linux, OS X
    - [Windows](windows.md)
* Importing to Java keystores
* Creating a CA cert and generating certificates for TLS