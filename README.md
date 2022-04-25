<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/session-hijacking/main/session-hijacking.png"></p>

An adversary may access the user's account using a stolen or leaked valid (existing) session identifier.

## Example #1
1. Adversary sniffs network traffic and gets a session identifier
2. Adversary uses the same session identifier to gain unauthorized access to bob's account

## Impact
Vary

## Risk
- gain unauthorized access

## Redemption
- identity confirmation
- regenerate session ids at authentication
- timeout and replace old session ids
- store ids in HTTP cookies

## ID
3693c458-c1b8-439f-8f0b-c3620c1c0129

## References
- [wiki](https://en.wikipedia.org/wiki/session_hijacking)
