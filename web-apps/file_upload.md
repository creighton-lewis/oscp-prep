# How To Bypass Client Validation
- Use BURPSuite to change the content-type submitted in request
- Manipulating front end code 
# Bypassing Blacklist Filters 
[Php Wordlist](https://raw.githubusercontent.com/Noredo/Wordlist-file-upload-extension-bypass/refs/heads/main/php.wordlist)
https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/web-extensions.txt

- Replace the end of the image or file name with this in burpsuite 

```
   for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

# Bypassing Type Filters 

## Content Type 

- Tricks website into thinking that a file that is not an image is an image

```
brainiac101@htb[/htb]$ wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/web-all-content-types.txt brainiac101@htb[/htb]$ cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```

## MIME Type 


```
echo "GIF8" > text.jpg brainiac101@htb[/htb]$ file text.jpg text.jpg: GIF image data
```
