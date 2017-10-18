Windows
========

Creating a certificate
--------

The easiest way to create a certificate file on Windows is with Powershell.  You can create a .pfx file containing a self-signed certificate with the public and private keys, all ready for import into an IIS web server.

```powershell
New-SelfSignedCertificate -CertStoreLocation cert:\currentuser\my -DnsName someserver.example.com
```

That creates a certificate in the current user's personal store.  No need for elevated priviledges to do so.

```
Thumbprint                      Subject
----------                      --------
SOMELONGHEXSTRING               CN=someserver.example.com
```

To get this into a .pfx file you need to create a secure string with the password you want on the .pfx file (securing the private key), and then use that to export the certificate.

```powershell
$certpassword = ConvertTo-SecureString -String "1234" -Force -AsPlainText
Export-PfxCertificate -cert cert:\currentuser\my\SOMELONGHEXSTRING -FilePath C:\Place\To\Save\Cert\someserver.example.com.pfx -Password $certpassword
```

This exports the public and private key and saves to a .pfx file on the filesystem, and you can take it to your web server and import it.

Now on your web server, with elevated priviledges (Powershell > Run as Administrator) you can import the certificate to the machine store, which makes it available to IIS:

```powershell
Import-PfxCertificate -FilePath C:\Place\To\Save\Cert\someserver.example.com.pfx -cert cert:\localmachine\my -Password $certpassword
```

Of course, you if you can just generate the file directly on the web server, you can cut out the import/export steps and, with Powershell running as an administtor:

```powershell
New-SelfSignedCertificate -CertStoreLocation cert:\localmachine\my -DnsName server.example.com
```

In either case, you have a certificate in the machine store, ready for IIS to use.  Now create a site (or pick an existing one) and add an HTTPS binding using this new certificate:

```powershell
mkdir c:\inetpub\someserver.example.com
New-Website -Name "someserver.example.com" -PhysicalPath c:\inetpub\someserver.example.com -Port 443 -Ssl

cd IIS:\SslBindings
Get-Item cert:\localmachine\my\SOMELONGHEXSTRING | New-Item *!443
```

Now we have a new website in IIS listening on all interfaces, port 443 (default for HTTPS), and we created a binding from the certificate in the localmachine store with the thumbprint `SOMELONGHEXSTRING` to all interfaces, port 443.

Trusting the certificate
--------

On Windows, you have to do two things to trust the certificate.  Being in your machine store isn't enough - it has to be in the Trusted Root Certificates store.

```
Import-PfxCertificate -FilePath C:\Place\To\Save\Cert\someserver.example.com.pfx -cert cert:\currentuser\root -Password $certpassword
```
You'll get a dialog verifying that you want to import this certificate into the trusted store, showing you the thumbprint.

Next, we need the hostname to match with the common name on the certificate.

```powershell
Add-Content C:\Windows\System32\drivers\etc\hosts "ip.address.of.server    someserver.example.com"
```

You can use 127.0.0.1 if your just hitting your local machine, or if you installed the certificate and site on a remote server, use it's IP address.  Now you can connect with a browser or .NET application and it will magically trust your self-signed certificate.