# Responsible Vulnerability Disclosure

Format taken from https://www.facebook.com/whitehat/report/

## Vendor

[Snyk](https://snyk.io)

## Product

[Snyk Advisor](https://snyk.io/advisor/)

## Vulnerability Type

Stored XSS on `snyk.io` domain

## Title

Markdown sanitization of README.md files of npm packages displayed by Snyk Advisor is lacking, resulting in a potential for storing an XSS exploit

## Description

Markdown format accepts HTML tags by default according to the official Markdown specification. 
However, HTML tags can result in JavaScript code execution. 
Therefore, when parsing Markdown content into HTML and integrating it into a web application, it can result in XSS.
This is the case for Snyk's Advisor service which displays Markdown content of differet npm packages which they control without any protection against XSS.

## Impact

[Medium CVSS score](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator?vector=AV:N/AC:L/PR:H/UI:R/S:U/C:N/I:H/A:N&version=3.1) and potentially higher.

### Proven impact

As thoroughly described and demonstrated in the complimentary [blog post](../blogpost/README.md), 
an attacker can use this XSS to completely fabricate the health score granted by the Advisor and make a package they control seem 
legit and widely addopted when in fact that's far from being true. 

Consider the following steps:

1. Attacker creates a new npm package called `png2jpg`.
2. Attacker edits the README of the package to look legit but also injects an XSS payload.
3. Snyk Advisor scans the new package and displays it at https://snyk.io/advisor/npm-package/png2jpg
4. Attacker publishes a new version with a more advanced payload, that when is executed in the https://snyk.io/advisor/ app, changes its layout completely to pose as a healthy, highly maintained and widely addopted package (by simply modifying the HTML of the app on-the-fly)
5. Snyk Advisor scans the new version, so now whenever someone's visiting https://snyk.io/advisor/npm-package/png2jpg, the payload will load and the package will look legit.
6. Attacker promotes the package as a png-to-jpg converter service, specifically with the Snyk Advisor link, so that people will be convinced the package is good for use.
7. Since the Advisor layout confirms the package is legit, the victim will consider using it.
8. The fact the Advisor suggests to copy the `npm install png2jpg` command makes it worse - no reason for the victim to visit the npm package over at https://npmjs.com, where they could have found out the package is not what it appears to be by the advisor.
9. Victim installs the malicious package after counting on Snyk Advisor without knowing it is in fact far from being healthy, and is in fact malicious.

### Unproven impact

An XSS can also lead to compromising of the user under `snyk.io`:
* If the user is logged in, credentials are in danger.
* If the user is not logged in but sensitive information is collected and is used to identify them, that info is in danger.

I did not take the time to prove the existence of such impact.

## Reproduction steps

To the date of writing this (19/03/23), the exploit can be observed in action when visiting an npm package I (the reporter) control at https://snyk.io/advisor/npm-package/png2jpg.

To reproduce yourself:

1. Create a new local folder.
2. `npm init --y`.
3. Name the package a name that doesn't already exist as an npm package (e.g. `sdklnfdsnjkl948498djkw-3`) by editing the `package.json` file.
4. Copy paste the following file https://raw.githubusercontent.com/weizman/png2jpg/master/README.md as `README.md` in the folder.
5. Publish the package.
6. After Snyk scan, visit https://snyk.io/advisor/npm-package/<PACKAGE_NAME>/.
7. You will see an alert message which proves Stored XSS on `snyk.io`.

## Resources

1. [contents of `curl https://snyk.io/advisor/npm-package/png2jpg`](./snyk-io-advisor-npm-package-png2jpg.html) to date 19/03/23.
2. [blogpost](../blogpost/) explaining the research process and the potential damage thoroughly.
3. [demo package](../demo/) I used to exploit the vulnerability.

## Recommendations

1. Make sure to sanitize any exnternal content from XSSable HTML tags, including Markdown content (GitHub/npm are great examples for excellent sanitazation)
2. Integrate CSP to disallow any unwanted JavaScript code execution on `snyk.io` (Similar to how [Socket.dev](http://socket.dev/npm/package/png2jpg) does this)

## Reporter

Gal Weizman @ https://weizman.github.io/
