<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/session-hijacking/main/content/session-hijacking.svg"></p>

An application manages session identifiers insecurely, making them vulnerable to interception by threat actors. Session identifiers are essential for maintaining a user's authenticated session. If these identifiers are transmitted, stored, or handled improperly, a threat actor may capture a valid session ID through methods like network interception or session theft. If a session identifier is compromised, the threat actor could use it to impersonate the legitimate user.

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