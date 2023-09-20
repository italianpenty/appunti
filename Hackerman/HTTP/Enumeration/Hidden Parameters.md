Potrebbero esserci nascosti parametri che non vedi tra le pagine.
Puoi provare a cercarli facendo due tipi di richieste : `GET` e `POST`
### **GET***
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u <URL>?FUZZ=key
```

### **ti**