Potrebbero esserci nascosti parametri che non vedi tra le pagine.
Puoi provare a cercarli facendo due tipi di richieste : `GET` e `POST`
### **GET***
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u <URL>?FUZZ=key
```

### **POST**
```
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u <URL> -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded'
```





I