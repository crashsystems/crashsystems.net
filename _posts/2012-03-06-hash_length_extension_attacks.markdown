---
layout: post
title: Hash Length Extension Attacks
aside: How to attack message authentication codes
tags:
- cryptography
- crypto
- hash Functions
- hash length extension
- hash message extension
---

It seems that many penetration testers do not do much to test cryptographic vulnerabilities. I've always been interested in cryptography, so I've made it a goal of mine to understand how web application developers misuse crypto, and how to exploit those flaws.

In January I had some time at work to do some independent research, so I decided to figure out how to perform hash length extension attacks against poorly implemented message authentication codes (MACs). I found several good [research papers](http://netifera.com/research/flickr_api_signature_forgery.pdf) and blog posts discussing how these attacks work in a very general sense, but not much that explained what was going on under the hood. In this post, I'll be explaining what is happening in a length extension attack.

###Message Authentication Codes 101
Message authentication codes (MACs) are a way to verify the authenticity of a message. In the more naive implementation of a MAC, the server has a secret key that it concatenates with a message, and then hashes the combination with an algorithm such as MD5 or SHA1. Take for example, an application that is designed to give a user the ability to download specific files, so long as the user has authorization to do so. The site may create a MAC for the filename like this:

<pre class="code hardWrap">def create_mac(key, fileName)
	return Digest::SHA1.hexdigest(key + fileName)
end</pre>

The resulting URL might look something like:

<pre class="code hardWrap">http://example.com/download?file=report.pdf&mac=563162c9c71a17367d44c165b84b85ab59d036f9</pre>

When a user sends the request to download a file, the following function is executed:

<pre class="code">def verify_mac(key, fileName, userMac)
	validMac = create_mac(key, filename)
	if (validMac == userMac) do
		initiateDownload()
	else
		displayError()
	end
end</pre>

With this code, the server should only call initiateDownload if the user has not tampered with the filename... or so the theory goes. In reality, this method of creating a MAC leaves the site vulnerable to an attack where an attacker can append their own content to the end of the file parameter.

###Length Extension Attacks, The Simple Explanation
Cryptographic hash functions such as MD5, SHA1, SHA2 etc are based upon a construct known as [Merkle–Damgård](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction). An interesting issue with this type of hash function is that if you have a message that was concatenated with some secret and the resulting hash of the concatenated value (the MAC), and you know the length of the secret, you can add your own data to the message and calculate a value that will pass the MAC check, without knowing the secret.

Example:
<pre class="code">message + padding + extension</pre>

Continuing the example from above, an extension attack against the hypothetical file download MAC would look like this:

<pre class="code hardWrap">http://example.com/download?file=report.pdf%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%A8/../../../../../../../etc/passwd&mac=ee40aa8ec0cfafb7e2ec4de20943b673968857a5</pre>

###Length Extensions In Depth

To understand why this attack works, you must first understand some of what happens inside a hash function.

####How Hash Algorithms Work

Hash functions work on blocks of data. 512 bits is the block length for both MD5, SHA1 and SHA256. Most messages that are hashed don't have a length that is evenly divisible by a hash function block length, so the message must be padded to match a multiple of the block length. So with the file download MAC example, the message after padding would look like this (the 'x's represent the secret key):

<pre class="code hardWrap">xxxxxxxxxxxreport.pdf\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA8</pre>

In SHA1, the algorithm being used in this example, a hash consists of a series of five integers. When displayed, these integers are usually in hexadecimal format and concatenated together. The initial value, aka the registers, are set to this value when the algorithm is ran: 67452301, EFCDAB89, 98BADCFE, 10325476, C3D2E1F0. Then once the message has been padded, it is broken up into 512 bit blocks. The algorithm goes through these blocks and does a series of calculations with the blocks that update the registers. Once these calculations are completed, the contents of the registers are the resulting hash of the message.

####Calculating An Extension
The first step in calculating an extension is creating a new MAC. To do this, we must hash the contents of what we are extending the hash with, '/../../../../../../../etc/passwd' in our example. However, when we perform this hash, we must override the initial registers with the MAC from the original message (SHA1(key+message)). You can think of this as making our SHA1 function start off at the state where the server's hash function left off.

<pre class="code hardWrap">Attacker's MAC = SHA1(extension + padding) <- but with overridden registers</pre>

For this attack to work, the extension must be in it's own block when it goes into the server's hash function. The second step is to calculate enough padding such that key + message + padding == some multiple of 512 bits. In this example, the key is 11 characters long. Therefore, the padded message would look like this:

<pre class="code hardWrap">report.pdf\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA8</pre>

We then send our padded and extended message to the server, with our new MAC:

<pre class="code hardWrap">http://example.com/download?file=report.pdf%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%A8/../../../../../../../etc/passwd&mac=ee40aa8ec0cfafb7e2ec4de20943b673968857a5</pre>

What the server hashes when it hashes the attacker's hacked message is this: secret + message + padding to the next block + extension + padding to the end of the block. The result of the server's hash will be ee40aa8ec0cfafb7e2ec4de20943b673968857a5, which is what we got when we hashed our extension while overriding the registers with the original MAC, because we essentially started off with our hash where the server is at when it is half way through hashing out attack.

####How To Run The Attack
For simplicity, in this example I revealed that the key length was 11 characters. In a real world attack, an attacker won't know the length of the key beforehand. Therefore, the attacker needs a way to determine the key length.

Continuing the example, lets say that our vulnerable website returns different errors (HTTP response codes, error messages in a response body, etc) when a MAC validation fails vs when validation succeeded but the file was not found. An attacker can then calculate multiple extensions, one for each possible key length, and send each to the server. When the server responds with an error indicating the file was not found, then a length extension vulnerability has been found, and the attacker is free to calculate new extensions aimed at gaining unauthorized access to sensitive files on the server.

####How To Defend Against This Attack
The solution to this vulnerability is to use an algorithm known as [HMAC](https://en.wikipedia.org/wiki/HMAC). Instead of just hashing the key concatenated with the message, HMAC does something like this:

MAC = hash(key + hash(key + message))

How HMAC actually works is a bit more complicated, but that is the general idea. The important part is that since the key is hashed into the message twice, it is not vulnerable to the extension attack described in this post. HMAC was first published in 1996, and has since been implemented in just about every programming language's standard library.

###Summary
Though there are still some crazy people out there that write their own cryptographic algorithms, people have gradually gotten the idea that writing your own crypto is a bad idea. However, it is not enough to use publicly vetted crypto algorithms: you've got to use those algorithms in the right way. Unless you really know how the algorithms you are using work and how to use them *correctly*, it is better to rely on professionally vetted high level libraries that take care of the low level stuff for you.