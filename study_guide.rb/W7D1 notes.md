### Cookies
+ Know what `statelessness` means:
    - HTTP is stateless; each request that a client makes is independent of every other request.

        - "say we login to amazon, then fetch our personal wishlist. HTTP itself doesn't build in a way to tie the first request (the login) to the second request (fetch the wishlist). That means that, without additional work, our web app won't understand whose wishlist to display."
    
    - This is the problem that cookies were invented to solve.

+ Know how a cookie can store data on the client.
```
Request:
    POST /session HTTP/1.1
    Host: amazon.com

    user=ruggeri&password=gizmo_is_great

Response:
    HTTP/1.1 302
    Set-Cookie: user=ruggeri; session_token=135792468
    Location: http://www.amazon.com/
```
- The `Set-Cookie` line is telling the client (our web browser) to save some state in the browser: namely, the logged in Amazon user and a randomly generated session token.
- The browser is expected to upload this cookie as part of each subsequent request to `www.amazon.com`.
```
GET /wishlist http/1.1
Host: www.amazon.com
Cookie: user=ruggeri; session_token=135792468
```
- The way the server can know this new request is being made by user ruggeri is through the cookie.
-  It can verify the user is who they claim to be and that the cookie is not faked.
- When the user submitted the credentials, the session token set in the cookie would also be stored in the server's database. On future requests, we compare the session token for the specified user with the session token uploaded in the cookie.
- Without the cookie, the server would have no way to differentiate this request from user ruggeri from another request from user patel.
- `permanent cookie`: set to expire far in the future so that it continues to be uploaded.
- `session cookie`: no expiration date, cookie deleted when the web browser closes.
