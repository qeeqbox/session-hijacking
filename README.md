<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/session-hijacking/main/content/session-hijacking.svg"></p>

## Session Hijacking
Session hijacking is a security attack where an attacker gains unauthorized access to a user's active authenticated session by obtaining or abusing a valid session identifier, such as a session cookie or authentication token. Once the attacker controls a user's session, they can impersonate the legitimate user and perform actions within the application without knowing the user's password.

Session hijacking can lead to serious security consequences, including unauthorized access to sensitive information, financial fraud, account takeovers, and a loss of user trust.

## How Session Hijacking Works
1. User Authentication: A user logs into a web application. After successful authentication, the server creates a session and provides the user with a session identifier, commonly stored in a browser cookie. The browser automatically sends this session identifier with future requests, allowing the application to recognize the authenticated user.
2. Obtaining the Session Identifier: An attacker may obtain a valid session identifier through various techniques.
3. Using the Stolen Session: After obtaining a valid session identifier, the attacker can send requests to the web application using that identifier. The application may treat the attacker as the legitimate user because the session is valid. 

## Session Hijacking Techniques
- Cross-Site Scripting (XSS): XSS occurs when an attacker injects malicious JavaScript into a web page viewed by other users. If session cookies are not protected with the HttpOnly attribute, malicious scripts can access and steal session information. Even if cookies are protected with HttpOnly, XSS can still be dangerous because attackers may perform actions within the victim's active session.
- Man-in-the-Middle (MitM) Attacks: A Man-in-the-Middle attack occurs when an attacker intercepts communication between a user and a web application. If the application does not use HTTPS, an attacker may capture session identifiers transmitted over the network.
- Session Fixation: Session fixation occurs when an attacker forces a victim to use a session identifier that is already known to the attacker. After the victim authenticates, the attacker can use the known session identifier to access the user's account.
- Weak Session Management: Poor session management practices can make session hijacking easier.
- Malware or Browser Compromise: Malware installed on a user's device may access browser storage, session tokens, or authentication data. Attackers can then use stolen session information to impersonate the user.

## Impact of Session Hijacking
Successful session hijacking attacks can lead to:
- Account Takeover: Attackers may gain complete access to user accounts and perform actions as the victim.
- Data Exposure: Attackers may access confidential information stored within the user's account.
- Financial Fraud: Attackers may carry out unauthorized purchases, transfers, or other financial actions.
- Reputation Damage: Organizations affected by session hijacking incidents may experience financial losses, regulatory consequences, and a decline in customer trust.

## Mitigation Session Hijacking Strategies
To prevent Session Hijacking:
- Use Secure Cookie Settings: Configure authentication cookies with appropriate security attributes.
    - Secure: Sends cookies only over HTTPS connections.
    - HttpOnly: Prevents JavaScript from directly accessing cookies.
    - SameSite: Reduces the risk of cross-site request attacks such as CSRF.
- Use Strong Session Identifiers: Session identifiers should:
  - Generated using cryptographically secure random number generators.
  - Contain sufficient randomness and entropy.
  - Never contain predictable information.
- Regenerate Session IDs After Authentication: Applications should create a new session identifier after:
  - User login
  - Privilege escalation
  - Password changes
  - Sensitive account operations
- Implement Session Expiration and Revocation
  - Set reasonable session expiration times.
  - Use idle timeouts.
  - Invalidate sessions after logout.
  - Allow users to view and terminate active sessions


## Session Hijacking Example
Clone this current repo recursively
```sh
git clone --recurse-submodules https://github.com/qeeqbox/session-hijacking
```
Run the webapp using Python
```sh
python3 session-hijacking/vulnerable-web-app/webapp.py
```
Open the webapp in your browser 127.0.0.1:5142
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/session-hijacking/main/content/1.png"></p>
Use the default credentials (username: admin and password: admin) to login
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/session-hijacking/main/content/2.png"></p>
Open the Storage tab in the developer tools to examine the request cookies
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/session-hijacking/main/content/3.png"></p>

## Code
The application generates cookies insecurely using the gen_cookie() function. The cookies do not validate the request's origin and store values unencrypted.
```py
    def gen_cookie(self, row, max_age):
        cookies = SimpleCookie(self.headers.get('Cookie'))
        if 'session_id' in cookies:
            session_id = cookies['session_id'].value
        else:
            session_id = "".join(str(randint(1, 9)) for _ in range(5))
        #end_time = datetime.now() + timedelta(days=1)
        SESSIONS[session_id] = {"username":row[1], "department": row[3],"access":row[4], "is_admin":row[5]}
        cookie1 = SimpleCookie()
        cookie1['session_id'] = session_id
        cookie1['session_id']['path'] = '/'
        cookie1['session_id']['max-age'] = max_age
        cookie2 = SimpleCookie()
        cookie2['is_admin'] = row[5]
        cookie2['is_admin']['path'] = '/'
        cookie2['is_admin']['max-age'] = max_age
        cookie3 = SimpleCookie()
        cookie3['access'] = row[4]
        cookie3['access']['path'] = '/'
        cookie3['access']['max-age'] = max_age
        cookie4 = SimpleCookie()
        cookie4['department'] = row[3]
        cookie4['department']['path'] = '/'
        cookie4['department']['max-age'] = max_age
        cookies = [('Set-Cookie', cookie1.output(header='', sep='')),('Set-Cookie', cookie2.output(header='', sep='')),('Set-Cookie', cookie3.output(header='', sep='')),('Set-Cookie', cookie4.output(header='', sep=''))]
        return cookies
```
Functions like admin_only() that handle generated cookies do not validate their origins, which means they may accept potentially unauthorized or tampered cookies.
```py
    def admin_only(f):
        @wraps(f)
        def wrapper(self, *args, **kws):
            cookies = SimpleCookie(self.headers.get('Cookie'))
            if "is_admin" in cookies:
                if cookies['is_admin'].value == "1":
                    return f(self, *args, **kws)
            return b"Admin Privileges Needed"
        return wrapper
```
