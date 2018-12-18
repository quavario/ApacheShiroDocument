| 属性    | 类型   | 描述 |
| ------- | ------ | ---- |
| name    | string |      |
| value   | string |      |
| domain  | string |      |
| path    |        |      |
| expires |        |      |
| Max-age |        |      |
| size    |        |      |
| http    |        |      |
| secure  |        |      |



### Cookie attributes

In addition to a name and value, cookies can also have one or more attributes. Browsers do not include cookie attributes in requests to the server—they only send the cookie's name and value. Cookie attributes are used by browsers to determine when to delete a cookie, block a cookie or whether to send a cookie to the server.

#### Domain and path

The `Domain` and `Path` attributes define the scope of the cookie. They essentially tell the browser what website the cookie belongs to. For obvious security reasons, cookies can only be set on the current resource's top domain and its sub domains, and not for another domain and its sub domains. For example, the website `example.org` cannot set a cookie that has a domain of `foo.com` because this would allow the `example.org`website to control the cookies of `foo.com`.

If a cookie's `Domain` and `Path` attributes are not specified by the server, they default to the domain and path of the resource that was requested.[[36\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-36) However, in most browsers there is a difference between a cookie set from `foo.com` without a domain, and a cookie set with the `foo.com` domain. In the former case, the cookie will only be sent for requests to `foo.com`, also known as a host-only cookie. In the latter case, all sub domains are also included (for example, `docs.foo.com`).[[37\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-37)[[38\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-38) A notable exception to this general rule is Edge prior to Windows 10 RS3 and Internet Explorer prior to IE 11 and Windows 10 RS4 (April 2018), which always send cookies to sub domains regardless of whether the cookie was set with or without a domain.[[39\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-39)

Below is an example of some `Set-Cookie` HTTP response headers that are sent from a website after a user logged in. The HTTP request was sent to a webpage within the `docs.foo.com` subdomain:

```
HTTP/1.0 200 OK
Set-Cookie: LSID=DQAAAK…Eaem_vYg; Path=/accounts; Expires=Wed, 13 Jan 2021 22:23:01 GMT; Secure; HttpOnly
Set-Cookie: HSID=AYQEVn…DKrdst; Domain=.foo.com; Path=/; Expires=Wed, 13 Jan 2021 22:23:01 GMT; HttpOnly
Set-Cookie: SSID=Ap4P…GTEq; Domain=foo.com; Path=/; Expires=Wed, 13 Jan 2021 22:23:01 GMT; Secure; HttpOnly
…
```

The first cookie, `LSID`, has no `Domain` attribute, and has a `Path` attribute set to `/accounts`. This tells the browser to use the cookie only when requesting pages contained in `docs.foo.com/accounts` (the domain is derived from the request domain). The other two cookies, `HSID` and `SSID`, would be used when the browser requests any subdomain in `.foo.com` on any path (for example `www.foo.com/bar`). The prepending dot is optional in recent standards, but can be added for compatibility with [RFC 2109](https://tools.ietf.org/html/rfc2109) based implementations.[[40\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-40)

#### Expires and Max-Age

The `Expires` attribute defines a specific date and time for when the browser should delete the cookie. The date and time are specified in the form `Wdy, DD Mon YYYY HH:MM:SS GMT`, or in the form `Wdy, DD Mon YY HH:MM:SS GMT` for values of YY where YY is greater than or equal to 0 and less than or equal to 69.[[41\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-41)

Alternatively, the `Max-Age` attribute can be used to set the cookie's expiration as an interval of seconds in the future, relative to the time the browser received the cookie. Below is an example of three `Set-Cookie`headers that were received from a website after a user logged in:

```
HTTP/1.0 200 OK
Set-Cookie: lu=Rg3vHJZnehYLjVg7qi3bZjzg; Expires=Tue, 15 Jan 2013 21:47:38 GMT; Path=/; Domain=.example.com; HttpOnly
Set-Cookie: made_write_conn=1295214458; Path=/; Domain=.example.com
Set-Cookie: reg_fb_gate=deleted; Expires=Thu, 01 Jan 1970 00:00:01 GMT; Path=/; Domain=.example.com; HttpOnly
```

The first cookie, `lu`, is set to expire sometime on 15 January 2013. It will be used by the client browser until that time. The second cookie, `made_write_conn`, does not have an expiration date, making it a session cookie. It will be deleted after the user closes their browser. The third cookie, `reg_fb_gate`, has its value changed to "deleted", with an expiration time in the past. The browser will delete this cookie right away because its expiration time is in the past. Note that cookie will only be deleted if the domain and path attributes in the `Set-Cookie` field match the values used when the cookie was created.

As of 2016 Internet Explorer did not support `Max-Age`.[[42\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-42)[[43\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-43)

#### Secure and HttpOnly

The `Secure` and `HttpOnly` attributes do not have associated values. Rather, the presence of just their attribute names indicates that their behaviors should be enabled.

The `Secure` attribute is meant to keep cookie communication limited to encrypted transmission, directing browsers to use cookies only via [secure/encrypted](https://en.wikipedia.org/wiki/HTTPS) connections. However, if a web server sets a cookie with a secure attribute from a non-secure connection, the cookie can still be intercepted when it is sent to the user by [man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack). Therefore, for maximum security, cookies with the Secure attribute should only be set over a secure connection.

The `HttpOnly` attribute directs browsers not to expose cookies through channels other than HTTP (and HTTPS) requests. This means that the cookie cannot be accessed via client-side scripting languages (notably [JavaScript](https://en.wikipedia.org/wiki/JavaScript)), and therefore cannot be stolen easily via [cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting) (a pervasive attack technique).[[44\]](https://en.wikipedia.org/wiki/HTTP_cookie#cite_note-Symantec-2007-2nd-exec-44)