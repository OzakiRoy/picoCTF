
```
	function send_request() {
		let val = document.getElementById("name").value;
		// ^p.....F!?
		fetch(`/flag?input=${val}`)
			.then(res => res.text())
			.then(res => {
				const res_json = JSON.parse(res);
				alert(res_json.flag)
				return false;
			})
		return false;
	}
```

Starting "p" and ends with "F"

```1
▶ curl -XPOST http://saturn.picoctf.net:64513/flag\?input=picoCTF   
{"flag":"picoCTF{succ3ssfully_matchtheregex_0694f25b}"}%  
```
