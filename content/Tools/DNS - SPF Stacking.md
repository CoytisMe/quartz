---
category: troubleshooting
tags:
  - dns
  - spf
  - dkim
  - email
  - deliverability
  - zoho
---
https://support.greenhood.com.au/a/tickets/182936

### Zoho's Instructions for Domain Verification:

1. Sign in to your domain host.
2. Go to the DNS records page of your domain website and locate the option to add TXT records.
3. Add all the TXT records given below one by one. 
   (zoho-verification=zb10057179.zmverify.zoho.com.au)
   For hostname put @ or leave it blank. 
   **It wouldn't let me so I put the domain name**
4. You may have to wait 24-48 hours for the values to propagate, depending on the TTL value you entered.
5. Later, return to this page and click Validate Records to complete the verification process.

**This worked**

### SPF and DKIM
Given a record for each, what I thought to do was correct.

### Paddy's notes on SPF.

Well done in adding the required entry to the existing SPF!

However... what happens when we add extra entries to SPF record is it can tip it over the 10 lookup record and break it, see below from dmarcian

![[Pasted image 20251128211450.png]]

Silly one.zoho.com.au adds 5 lookups and takes the total from 9 to 14 and makes the SPF record invalid, which is a stupid shortcoming of the interwebs.
When adding to an existing record, the go is to run an SPF validation afterwards to see if you have overstuffed it.

**How to fix:**
option 1 - DNS flattening - this is tedious and not best practice and a story for another day
option 2 - audit the record and be brutal with what stays. In this case, if you look at the SPF survey in the link above, if i stop it validating email sent from our web server (typically from the website's contact form to mark's email address i.e website enquiries) I can free up enough.
ip4:103.27.33.24 - our hosting server
include:mxs.au - our hosting server
+a - the A record showing the IP of the server: +a:adjusting.com.au:103.27.33.24/32
include:spf.vps.hostingplatform.net.au: our hosting server

Yep it is a lot to know, but you recognise certain IP's and hostnames after a while.

I can remove these once I have made sure that the adjusting,com.au web form is set to use a connector like smpt2go.
Once I do that, I can safely remove the extra SPF entries and hopefully get down under 10.
I will attend to this bit. 

Please make this into a KB.

You're doing a good job though, there is a lot to know with this stuff and I have never found one single resource that is helpful.