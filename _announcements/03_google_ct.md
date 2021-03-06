---
layout: default
navtitle: Chrome Certificate Transparency (CT) Impact
title: Chrome Certificate Transparency Requirements
pubDate: May 10, 2018
collection: announcements
permalink: announcements/chromect/
description:  Upcoming changes to Chrome could affect your agency. This change requires all TLS/SSL certificates to appear in a CT log when they validate to a Root CA certificate distributed through an Operating System (OS) trust store. The Microsoft and Apple Trust Stores currently distribute the U.S. Government Root CA (Federal Common Policy CA) certificate. This change will take effect starting with **Chrome 68** and will affect any TLS/SSL certificate issued after **April 30, 2018.**<br><br>
---

{% include alert-info.html content="At this time, the Federal PKI Certification Authorities used by most federal agencies for intranet TLS/SSL certificates do not support Certificate Transparency logging requirements." %}  

Upcoming changes to Chrome could impact your agency. Google will start enforcing Certificate Transparency (CT) with **Chrome 68** for all TLS/SSL certificates issued after April 30, 2018. This means that all TLS/SSL certificates issued after April 30, 2018, that validate to a publicly trusted Root Certification Authority (CA) certificate must appear in a CT log in order to be trusted in Chrome 68 and above. In addition, website operators must serve proof of the CT log inclusion (i.e., a signed certificate timestamp).

- [How Does This Work?](#how-does-this-work)
- [What Will Be Impacted?](#what-will-be-impacted)
- [When Will This Start?](#when-will-this-start)
- [What Should I Do?](#what-should-i-do)
- [How Can I Test?](#how-can-i-test-ct-compliance-for-my-intranet-website)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Additional Resources](#additional-resources)

{% include alert-info.html content="All popular browsers plan to deploy CT in their product roadmaps. A more complete timeline will be posted on this site as dates and deployment timelines become known." %}

## How Does This Work?

The requirements for CT are built into _browsers_. 

- All roots that have been distributed _by one or more_ of the Microsoft, Android, Apple, or Mozilla trusted root programs are listed here: [Root Stores](https://cs.chromium.org/chromium/src/net/data/ssl/root_stores/README.md){:target="_blank"}.
- When a government user browses to an intranet website, the user's workstation or mobile device will build one or more certificate paths to the enterprise or publicly trusted roots. 
- The browser will compare the certificate path(s) to the list of roots that have _ever_ been included in the popular trust stores currently in use worldwide.
- If any certificate in the trust chain matches one of the roots in the list, then the CT requirements will be in effect. 

The Microsoft and Apple trust stores currently distribute the U.S. Government Root CA (Federal Common Policy CA) certificate.

## What Will Be Impacted?

A government user will receive an error on government-furnished equipment if all of the following are true: 

1. Using Chrome 68 or higher (Additional browsers may be affected in the future)
2. Browsing to an intranet website with a TLS/SSL certificate that validates to the Federal Common Policy CA
3. The TLS/SSL certificate was issued after **April 30, 2018**

![Chrome Error Screen]({{site.baseurl}}/img/google_ct_hot_topic_error.png){:style="width:55%;float:center;"}

## When Will This Start?
Google will enforce CT starting with Chrome 68. _Estimated_ release dates are:
- Chrome 68 Beta:  June 7, 2018
- Chrome 68 Stable:  July 24, 2018

## What Should I Do?
The Federal PKI community has notified the Microsoft Trusted Root Program to remove the trust for TLS/SSL from the globally distributed Federal Common Policy CA.

To mitigate the impact on the federal enterprise:  

1. You should distribute the Federal Common Policy CA to government-furnished equipment as an _enterprise trusted root certificate_. 
2. You must disable CT enforcement for the affected intranet websites. 

Please see [Disable CT Enforcement for Government-Furnished Equipment](#disable-ct-enforcement-for-government-furnished-equipment).


### Disable CT Enforcement for Government-Furnished Equipment
{% include alert-info.html content="Two options are outlined in this section. Additional options may become available in future releases of Chrome. We will post more information as we update the procedures. Please also check the GitHub Issues in the GSA FPKI-Guides repository for any in-progress discussions." %} 

#### Option 1:&nbsp;&nbsp;Disable CT Enforcement for "Legacy" CAs (Recommended Configuration)

Google Chrome's "CertificateTransparencyEnforcementDisabledForLegacyCas" policy configuration allows you to disable CT enforcement for websites that chain to a user-specified "legacy" CA. Google Chrome categorizes a CA as "legacy" if it meets the following criteria:
1. The CA has been publicly trusted by default in one or more operating systems supported by Google Chrome, such as Windows or macOS.
2. The CA isn't currently trusted by the Android Open Source Project or Chrome OS.

The Federal Common Policy CA meets Google's criteria for a "legacy" CA, so you can disable CT enforcement for intranet websites that chain to it. In some cases, you'll need to create a new registry key tree in the locations specified below:

**a.&nbsp;&nbsp;Windows Registry location for Windows clients:**<br>

For _HKEY_LOCAL_MACHINE\Software\Policies\Google\Chrome\CertificateTransparencyEnforcementDisabledForLegacyCas_, add a new string value:
   
   ```
   Name = 1 | Data = sha256/jotW9ZGKJb2F3OdmY/2UzCNpDxDqlYZhMXHG+DeIkNU=
   ```
   
**b.&nbsp;&nbsp;Windows Registry location for Chrome OS clients:**<br>

For _HKEY_LOCAL_MACHINE\Software\Policies\Google\ChromeOS\CertificateTransparencyEnforcementDisabledForLegacyCas_, add new string value:

   ```
   Name = 1 | Data = sha256/jotW9ZGKJb2F3OdmY/2UzCNpDxDqlYZhMXHG+DeIkNU=
   ```
   
**c.&nbsp;&nbsp;macOS**<br>

For preference name, _CertificateTransparencyEnforcementDisabledForLegacyCas_, add values:

   ```
   <array>
     <string>sha256/jotW9ZGKJb2F3OdmY/2UzCNpDxDqlYZhMXHG+DeIkNU=</string>
   </array>
   ```

**Note:**&nbsp;&nbsp;In all cases above, `jotW9ZGKJb2F3OdmY/2UzCNpDxDqlYZhMXHG+DeIkNU=` is a base64 encoding of a SHA-256 hash of the Federal Common Policy CA's Subject Public Key Information (SPKI) field.


#### Option 2:&nbsp;&nbsp;Disable CT Enforcement for Domains and Sub-Domains

Enterprise Chrome for government-furnished equipment will not enforce CT requirements if you apply a policy rule and include a **.gov or .mil second-level domain**, such as _agency.gov_, or other **third-level sub-domains**, such as _example.agency.gov_. You should apply configuration changes for only government-furnished equipment and only include an explicit list of second-level or below sub-domains in use for intranet websites. In some cases, you may need to create a new registry key tree in the locations specified below: 


**a.&nbsp;&nbsp;Windows Registry location for Windows clients:**<br>

For _HKEY_LOCAL_MACHINE\Software\Policies\Google\Chrome\CertificateTransparencyEnforcementDisabledForUrls_, add new string value:

   ```
   Agency Sub-Domain example:
   Name = 1 | Data = example.agency.gov
   
   Gov/Mil Top-Level Domain example: 
   Name = 2 | Data = gov
   Name = 3 | Data = mil
   ```
   
**b.&nbsp;&nbsp;Windows Registry location for Google Chrome OS clients:**<br>

For _HKEY_LOCAL_MACHINE\Software\Policies\Google\ChromeOS\CertificateTransparencyEnforcementDisabledForUrls_, add new string value:

   ```
   Sub-Domain example:
   Name = 1 | Data = example.agency.gov
   
   Gov/Mil Top-Level Domain example: 
   Name = 2 | Data = gov
   Name = 3 | Data = mil
   ```
   
**c.&nbsp;&nbsp;macOS**<br>

For _preference name_, _CertificateTransparencyEnforcementDisabledForUrls_, add values:<br>

   ```
   <array>
     <string>example.agency.gov</string>
     <string>.example.agency.gov</string>
     <string>gov</string>
     <string>mil</string>
   </array>
   ```
   
## How Can I Test CT Compliance for My Intranet Website?
To test CT compliance, you'll need to use a pre-release version of Chrome. 

- Google will start enforcing CT with [Chrome 68](https://www.chromium.org/developers/calendar){:target="_blank"}. Chrome 68 is available for a limited time at [Chrome Canary channel](https://www.google.com/chrome/browser/canary.html){:target="_blank"}. Download and install it as recommended by Google.
- You'll need to use a special command line flag to execute the browser: [Add a command-line flag for CT testing](https://bugs.chromium.org/p/chromium/issues/detail?id=816543&can=2&q=816543&colspec=ID%20Pri%20M%20Stars%20ReleaseBlock%20Component%20Status%20Owner%20Summary%20OS%20Modified){:target="_blank"}.

**Important Testing Note:** CT enforcement and website certificate chain information is cached in Chrome. Before you start each test, clear the cached data from within the browser:<br>
   ```
   Settings->Advanced
   Ctrl + Shift + Del
   ```
   
1. Find the directory path to the new Chrome executable. For example: 

   ```
   Windows: C:\Users\<username>\AppData\Local\Google\Chrome SxS\Application\chrome.exe
   ```
   
2. Open a command line in the executable directory to enable CT enforcement for _a date in the past_, measured in seconds. For example, this command will enable CT enforcement for any TLS/SSL certificate issued after January 1, 2015 (1420086400 seconds):

   ```
   chrome.exe --enable-features="EnforceCTForNewCerts<EnforceCTTrial" --force-fieldtrials="EnforceCTTrial/Group1" --force-fieldtrial-params="EnforceCTTrial.Group1:date/1420086400"
   ```
   
3. Browse to one of your intranet websites protected by a TLS/SSL certificate issued from a Federal PKI CA to force a CT error.  Alternatively, you can use these sites: [Treasury PKI Homepage](https://pki.treas.gov/){:target="_blank"} or [Joint Personnel Adjudication System](https://jpasapp.dmdc.osd.mil/JPAS/JPASDisclosureServlet){:target="_blank"}.

4. If you don't see an error, clear the cache from the previous test and repeat Step 2 above.

5. Apply the registry settings given in [Disable CT Enforcement for Government-Furnished Equipment](#disable-ct-enforcement-for-government-furnished-equipment) for your intranet sites.

6. Clear the cache again, and re-launch Chrome using the command line argument in Step 2.

7. Observe the changes in CT enforcement errors and repeat as needed.

{% include alert-info.html content="Thank you to NASA teams for these testing procedures.  Please check the GitHub Issues in the GSA's fpki-guides Playbook repository for any in-progress discussions." %}

## Frequently Asked Questions

### 1. Will Google's use of CT in Chrome impact my agency's internal, only locally trusted CA TLS/SSL certificates?

No. There will be no impact if you use your agency's internal, only locally trusted CA to issue TLS/SSL certificates to intranet sites. Chrome's CT change will impact only those TLS/SSL certificates that validate to Federal Common Policy CA, whose certificate is distributed through the Microsoft and Apple trust stores.

### 2. Why is Google enforcing CT in Chrome?

Chrome's CT change has been planned and incrementally implemented for over two years.  CT provides a benefit to the global community by:

- Improving openness and transparency
- Allowing domain owners to identify mistakenly or maliciously issued certificates 

### 3. How do I know whether my intranet website is compliant with CT?
These procedures apply to any government internet or intranet website and any Federal PKI TLS/SSL certificate or commercially sourced certificate.

**Note:**&nbsp;&nbsp;Signed certificate timestamps (SCTs) are only required for certificates issued after April 30, 2018.  Some certificates issued **before** this date may already be compliant.  

1. Open Chrome and browse to your website.
2. In Chrome, go to **Settings->More Tools**.
3. Open the **Developer Tools** panel:<br>
   ```
   Windows:  CTRL + Shift + "i"
   macOS:  Apple key + Shift + "i"
   ```
4. Select the **Security** tab in the **Developer Tools**.
5. Refresh the website page and click on the website under the **Main origin** column.
6. If the certificate is compliant, it will display the CT log details.

## Additional Resources
1. [What is Certificate Transparency](https://www.certificate-transparency.org/){:target="_blank"}  
2. [Certificate Transparency--Resources for Site Owners](https://sites.google.com/site/certificatetransparency/resources-for-site-owners){:target="_blank"}    
3. [Certificate Transparency Announcement](https://groups.google.com/a/chromium.org/forum/#!topic/ct-policy/78N3SMcqUGw){:target="_blank"} 
4. [How to Disable CT in Enterprise Chrome](http://www.chromium.org/administrators/policy-list-3#CertificateTransparencyEnforcementDisabledForUrls){:target="_blank"}    
5. [Chrome Policy Templates](https://www.chromium.org/administrators/policy-templates){:target="_blank"}  
6. [Example of Valid CT Certificate Record in Chrome](https://www.certificate-transparency.org/certificate-transparency-in-chrome){:target="_blank"} 
7. [Command Line Flags for Certificate Transparency Testing](https://bugs.chromium.org/p/chromium/issues/detail?id=816543&can=2&q=816543&colspec=ID%20Pri%20M%20Stars%20ReleaseBlock%20Component%20Status%20Owner%20Summary%20OS%20Modified){:target="_blank"} 
