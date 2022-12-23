<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/session-hijacking/main/session-hijacking.png"></p>

A threat actor may access the user's account using a stolen or leaked valid (existing) session identifier.

## Example #1
1. Threat actor sniffs network traffic and gets a session identifier
2. Threat actor uses the same session identifier to gain unauthorized access to bob's account

## Impact
Vary

## Risk
- Gain unauthorized access

## Redemption
- Identity confirmation
- Regenerate session ids at authentication
- Timeout and replace old session ids
- Store ids in HTTP cookies

## ID
3693c458-c1b8-439f-8f0b-c3620c1c0129

## References
- [wiki](https://en.wikipedia.org/wiki/session_hijacking)
