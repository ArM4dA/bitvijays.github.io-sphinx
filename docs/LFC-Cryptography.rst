Learning from the CTF : Cryptography
====================================

This post (Work in Progress) lists the tips and tricks while doing Cryptography challenges during various CTF’s.

* Caesar cipher and substitution cipher can be solved by using Cryptool 1. Just check the Analysis option, there’s analysis for Symmetric Key,Asymmetric Key, Hash and others. Otherwise, a good website to solve substitution cipher is  `Quipqiup <http://quipqiup.com/>`_. Ceaser cipher good website is `Caesar cipher decryption tool <https://www.xarg.org/tools/caesar-cipher/>`_

* If you get some text atleast few paragraphs and some random numbers as (1, 9, 4) (4, 2, 8) (4, 8, 3) (7, 1, 5) (8, 10, 1), it might mean (Paragraph, Line, Word). 

* If you get a poem followed by some numbers and ciphertext, probably it's `Poem Code <https://en.wikipedia.org/wiki/Poem_code>`_ The poem code is a simple, and insecure, cryptographic method which was used by SOE to communicate with their agents in Nazi-occupied Europe. The method works by the sender and receiver pre-arranging a poem to use.

* If you get some ciphertext encrypted by XOR, `xortool <https://github.com/hellman/xortool>`_. It can help you to find the key length and the key.

* All ciphers listed at `Cipher Tools <http://rumkin.com/tools/cipher/>`_

* If we have a "3845281945283805284526053525260547380516453748164748478317454508" something like this? It might be a  Polybius square cipher, where each 2-number block in the encrypted text is a coordinate on a 5x5 square.

* If we examine the encrypted string, we see that when divided into blocks of 2 every first letter, x, falls on the interval 0<=x<=4 and every second letter, y, falls on the interval 5<=y<=9. This knowledge combined with the hint that our cipher will be missing the letter "z", we can construct a 5x5 Polybius square (hence the hint sqrt(ABCDEFGHIJKLMNOPQRSTUVWXY), because the alphabet minus Z is of length 25 and sqrt(25) = 5).

* Provided with RSA p,q,dp,dq and ciphertext utilize `https://raw.githubusercontent.com/LFlare/picoctf_2017_writeup/master/cryptography/weird-rsa/solve.py>`_
 
